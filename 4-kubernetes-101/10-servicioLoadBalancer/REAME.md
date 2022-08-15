# Install Minikube multi node

Install minukube with two nodes from the scratch
```
minikube start --driver=kvm2 --cpus=2 --memory=2200mb --disk-size=15000mb --nodes=2 -p vcc
```

## minikube check commands
```
minikube -p vcc ip
minikube -p vcc status
minikube -p vcc profile list

minikube -p vcc delete

```

```
$ minikube -p vcc profile list                                                  
|---------|-----------|---------|----------------|------|---------|---------|-------|--------|
| Profile | VM Driver | Runtime |       IP       | Port | Version | Status  | Nodes | Active |
|---------|-----------|---------|----------------|------|---------|---------|-------|--------|
| vcc     | kvm2      | docker  | 192.168.39.169 | 8443 | v1.24.1 | Running |     2 | *      |
|---------|-----------|---------|----------------|------|---------|---------|-------|--------|
```

```
$ minikube -p vcc node list
vcc     192.168.39.169
vcc-m02 192.168.39.43                                                                  
```


```
minikube -p vcc --node=vcc ssh
minikube -p vcc --node=vcc-m02 ssh
```

## kubectl commands

```
$ kubectl get nodes
NAME      STATUS   ROLES           AGE    VERSION
vcc       Ready    control-plane   126m   v1.24.1
vcc-m02   Ready    <none>          125m   v1.24.1
```

```
$ kubectl get nodes -o wide 
NAME      STATUS   ROLES           AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE               KERNEL-VERSION   CONTAINER-RUNTIME
vcc       Ready    control-plane   126m   v1.24.1   192.168.39.169   <none>        Buildroot 2021.02.12   5.10.57          docker://20.10.16
vcc-m02   Ready    <none>          125m   v1.24.1   192.168.39.43   <none>        Buildroot 2021.02.12   5.10.57          docker://20.10.16
```

## Deployment Loadbalancer on Minikube
* replicaset --> `kubectl apply -f replicaset.yaml`
* loadbalancer --> `kubectl apply -f servicio_lb.yaml`

```
$ kubectl get all
NAME                                   READY   STATUS    RESTARTS   AGE                                                                                             
pod/nginx-testing-loadbalancer-r5cvg   1/1     Running   0          11m                                                                                             
pod/nginx-testing-loadbalancer-w89mv   1/1     Running   0          11m                                                            

NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE                          
service/kubernetes              ClusterIP      10.96.0.1        <none>        443/TCP        25m         
service/servicio-loadbalancer   LoadBalancer   10.111.166.245   <pending>     80:30207/TCP   4m39s                                           

NAME                                         DESIRED   CURRENT   READY   AGE                                                                 
replicaset.apps/nginx-testing-loadbalancer   2         2         2       11m                                                                     
```
Without cloud provider we'll install a load-balancer solution with `MetalLB`


## Install MetalLB

MetalLB is a load-balancer implementation for bare metal Kubernetes clusters, using standard routing protocols.

Kubernetes does not offer an implementation of network load balancers (Services of type LoadBalancer) for bare-metal clusters. The implementations of network load balancers that Kubernetes does ship with are all glue code that calls out to various IaaS platforms (GCP, AWS, Azure…).

If you’re not running on a supported IaaS platform (GCP, AWS, Azure…), LoadBalancers will remain in the `pending` state indefinitely when created.

### Add metallb addons on Minikube
List all addons available for minikube

```
minikube -p vcc addons list
|-----------------------------|---------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE |    STATUS    |           MAINTAINER           |
|-----------------------------|---------|--------------|--------------------------------|
| ambassador                  | vcc     | disabled     | 3rd party (Ambassador)         |
| auto-pause                  | vcc     | disabled     | Google                         |
| csi-hostpath-driver         | vcc     | disabled     | Kubernetes                     |
| dashboard                   | vcc     | disabled     | Kubernetes                     |
| default-storageclass        | vcc     | enabled ✅   | Kubernetes                     |
| efk                         | vcc     | disabled     | 3rd party (Elastic)            |
| freshpod                    | vcc     | disabled     | Google                         |
| gcp-auth                    | vcc     | disabled     | Google                         |
| gvisor                      | vcc     | disabled     | Google                         |
| headlamp                    | vcc     | disabled     | kinvolk.io                     |
| helm-tiller                 | vcc     | disabled     | 3rd party (Helm)               |
| inaccel                     | vcc     | disabled     | InAccel <info@inaccel.com>     |
| ingress                     | vcc     | disabled     | 3rd party (unknown)            |
| ingress-dns                 | vcc     | disabled     | Google                         |
| istio                       | vcc     | disabled     | 3rd party (Istio)              |
| istio-provisioner           | vcc     | disabled     | 3rd party (Istio)              |
| kong                        | vcc     | disabled     | 3rd party (Kong HQ)            |
| kubevirt                    | vcc     | disabled     | 3rd party (KubeVirt)           |
| logviewer                   | vcc     | disabled     | 3rd party (unknown)            |
| metallb                     | vcc     | enabled ✅   | 3rd party (MetalLB)            |
| metrics-server              | vcc     | disabled     | Kubernetes                     |
| nvidia-driver-installer     | vcc     | disabled     | Google                         |
| nvidia-gpu-device-plugin    | vcc     | disabled     | 3rd party (Nvidia)             |
| olm                         | vcc     | disabled     | 3rd party (Operator Framework) |
| pod-security-policy         | vcc     | disabled     | 3rd party (unknown)            |
| portainer                   | vcc     | disabled     | Portainer.io                   |
| registry                    | vcc     | disabled     | Google                         |
| registry-aliases            | vcc     | disabled     | 3rd party (unknown)            |
| registry-creds              | vcc     | disabled     | 3rd party (UPMC Enterprises)   |
| storage-provisioner         | vcc     | enabled ✅   | Google                         |
| storage-provisioner-gluster | vcc     | disabled     | 3rd party (unknown)            |
| volumesnapshots             | vcc     | disabled     | Kubernetes                     |
|-----------------------------|---------|--------------|--------------------------------|

```

### Enable metallb

```
minikube -p vcc addons enable metallb
```
Get all namespaces:

```
diego@thinkpad:~$ kubectl get ns
NAME              STATUS   AGE
default           Active   136m
kube-node-lease   Active   136m
kube-public       Active   136m
kube-system       Active   136m
metallb-system    Active   56m
```
Get all pods running in `metallb-system`

```
diego@thinkpad:~$ kubectl -n metallb-system get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/controller-6f655c76ff-xlp4k   1/1     Running   0          56m
pod/speaker-28cxx                 1/1     Running   0          56m
pod/speaker-jsk5h                 1/1     Running   0          56m

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
daemonset.apps/speaker   2         2         2       2            2           beta.kubernetes.io/os=linux   56m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           56m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-6f655c76ff   1         1         1       56m

```

* Configure addon

```
kubectl get configmap/config -n metallb-system -o yaml

$ kubectl -n metallb-system get configmaps config -o yaml
apiVersion: v1
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - -
....
...
```

If configmap is not configure, we must setup the IP's of the minikube cluster (all nodes)
* Get all IP's --> `kubectl get nodes -o wide`
* From minikube --> `minikube -p vcc node list`


```
$ minikube -p vcc addons configure metallb
-- Enter Load Balancer Start IP: 192.168.39.43
-- Enter Load Balancer End IP: 192.168.39.169 
  - Using image metallb/controller:v0.9.6                                                                
  - Using image metallb/speaker:v0.9.6                                                                                                       
* metallb was successfully configured                                                   
```

Now the service load-balancer should change the state `pending`

```
$ kubectl get all
NAME                                   READY   STATUS    RESTARTS   AGE                                                          
pod/nginx-testing-loadbalancer-r5cvg   1/1     Running   0          23m                                                 
pod/nginx-testing-loadbalancer-w89mv   1/1     Running   0          23m                                                 

NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE           
service/kubernetes              ClusterIP      10.96.0.1        <none>          443/TCP        37m         
service/servicio-loadbalancer   LoadBalancer   10.111.166.245   192.168.39.43   80:30207/TCP   16m                                             

NAME                                         DESIRED   CURRENT   READY   AGE                                 
replicaset.apps/nginx-testing-loadbalancer   2         2         2       23m           
```

It worked !

### Testing MetalLB in both pods

Now simple curl request to the local load-balancer will resolve in both pods. Let's login in each pod and change the nginx index.html page for something more useful.

Pod 1:
```
kubectl exec -it nginx-testing-loadbalancer-r5cvg bash
echo "servicio 1" > /usr/share/nginx/html/index.html
```

Pod 2:
```
kubectl exec -it nginx-testing-loadbalancer-w89mv bash
echo "servicio 2" > /usr/share/nginx/html/index.html
```

Finally:

```
$ curl 192.168.39.43
servicio 2

$ curl 192.168.39.43
servicio 1

$ curl 192.168.39.43
servicio 2

$ curl 192.168.39.43
servicio 1
```
