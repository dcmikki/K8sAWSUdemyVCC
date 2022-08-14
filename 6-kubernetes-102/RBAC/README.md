# Autentificación en Kubernetes

## Mas comunes
* Bearer Tokens
* Proxies de autentificacion
* HTTP basic auth (user/password)
* Certificados de client x509

## Minikube Certificados x509

* Hay un certificado generado para usuario
* Todos los certificados están firmados por un CA (tiene parte publica y privada)
* La comunicación va encriptada y Kubernetes comprueba si esta firmada por el CA generado al crear el clúster

### Location

```
$ ls ~/.minikube
...
ca.key
ca.crt
ca.pem
...
```

## Configurar Nuevo Usuario con acceso al clúster

Ejemplo con `vagrant` user de otra VM

```
vagrant@automation:~$ kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
vagrant@automation:~$
```

### Crear kubectl configuration

Crear directorio
```
vagrant@automation:~$ mkdir ~/.kube
```

Fichero configuracion para usurio `vagrant`

```
vagrant@automation:~$ cat ~/.kube/config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/vagrant/.kube/ca.crt
    extensions:
    - extension:
        provider: minikube.sigs.k8s.io
        version: v1.26.0
      name: cluster_info
    server: https://192.168.39.89:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        provider: minikube.sigs.k8s.io
        version: v1.26.0
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/vagrant/.kube/vagrant.crt
    client-key: /home/vagrant/.kube/vagrant.key
```

```
vagrant@automation:~$ kubectl get nodes
Error in configuration: 
* unable to read client-cert /home/vagrant/.kube/vagrant.crt for minikube due to open /home/vagrant/.kube/vagrant.crt: no such file or directory
* unable to read client-key /home/vagrant/.kube/vagrant.key for minikube due to open /home/vagrant/.kube/vagrant.key: no such file or directory
* unable to read certificate-authority /home/vagrant/.kube/ca.crt for minikube due to open /home/vagrant/.kube/ca.crt: no such file or directory
```

Hay que generar todos los ficheros con openssl

```
/home/vagrant/.kube/vagrant.crt
/home/vagrant/.kube/vagrant.key
/home/vagrant/.kube/ca.crt
```

NOTA: El fichero `ca.crt` corresponde con la parte publica de CA de Minikube en `~/.minikube/ca.crt`


### OpenSSL comandos desde Minikube host

#### Generar un random seed para openssl

Permite generar claves publicas privadas con openssl

```
diego@thinkpad:~$ openssl rand -out /home/diego/.rnd -hex 256
```

#### Generar `vagrant.key`

Crear directorio para usuarios

```
diego@thinkpad:~$ mkdir -p usuarios/vagrant
diego@thinkpad:~$ cd usuarios/vagrant
diego@thinkpad:~/usuarios/vagrant$ openssl genrsa -out vagrant.key 2048
diego@thinkpad:~/usuarios/vagrant$ ls
diego@thinkpad:~/usuarios/vagrant$ vagrant.key
``` 

#### Certificate signing request (CSR) `vagrant.csr`
Usando `vagrant.key` de antes se genera un CSR para la comunicación con el clúster con una configuración especifica. Hay que asignar un “Common Name” y grupo al hacer la petición:

```
diego@thinkpad:~/usuarios/vagrant$ openssl req -new -key vagrant.key -out vagrant.csr -subj "/CN=vagrant/O=developers"

diego@thinkpad:~/usuarios/vagrant$ ls
vagrant.csr  vagrant.key
```

#### Generar `vagrant.crt`
El CSR anterior se firma con el `CA` de Minikube (parte publica y privada de CA) y genera el fichero `vagrant.crt`.

```
diego@thinkpad:~/usuarios/vagrant$ openssl x509 -req -in vagrant.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out vagrant.crt -days 500
Signature ok
subject=CN = vagrant, O = developers
Getting CA Private Key

diego@thinkpad:~/usuarios/vagrant$ ls
vagrant.crt  vagrant.csr  vagrant.key
```

Se envían los ficheros generados al client (vagrant box) y se ejecuta de nuevo kubectl.

```
vagrant@automation:~/.kube$ kubectl get pods

Error from server (Forbidden): pods is forbidden: User "vagrant" cannot list resource "pods" in API group "" in the namespace "default"
```

La conexión del usuario al clúster ha sido satisfactoria. El nuevo error es porque no hay permisos para listar los recursos tipo `pods` en el default namespace


#### Importante
* Autentificación x509 --> done
* Permisos con roles via `RBAC`.


## RBAC (Role-based access control)
Los `RBAC` permiten a los usuarios ejecutar tareas después de autentificarse (x509) en el clúster.


### rbac-role.yaml
Define un tipo de permiso para el grupo `developers` con unas acciones sobre ciertos recursos. Aquí solo se le asignan permisos de lectura al los pods del `default` namespace.

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: developers
rules:
- apiGroups: [""]   # indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
### rbac-rolebinding.yaml
Asigna al usurio `vagrant` el role del grupo `developers` creado antes

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: developers
subjects:
- kind: User
  name: vagrant     # name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role        # this must be Role or ClusterRole
  name: developers  # must match the name of the Role
  apiGroup: rbac.authorization.k8s.io
```

Aplicamos ambos ficheros:

```
$ kubectl apply -f rbac-role.yaml -f rbac-rolebinding.yaml    
```

Para testear si el usuario `vagrant` puede ver los pods, desplegamos un pod:
```
$ kubectl apply -f pod1.yaml 
pod/nginx created                                                          
```

Y desde la VM tratamos de lista los pods del `default` namespace

```
vagrant@automation:~/.kube$ kubectl get pods -o wide

NAME    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          2m    172.17.0.3   minikube   <none>           <none>
```

#### Probar borrar el pod

```
vagrant@automation:~/.kube$ kubectl delete pod nginx 

Error from server (Forbidden): pods "nginx" is forbidden: User "vagrant" cannot delete resource "pods" in API group "" in the namespace "default"
```

Como se esperaba el usuario `vagrant` del group `developers` no tiene permisos para borrar los pods. Si podría añadir el verbo `delete` al role para dar mas permisos, etc.
