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
## kubectl comands

```
$ kubectl get nodes
NAME      STATUS   ROLES           AGE    VERSION
vcc       Ready    control-plane   126m   v1.24.1
vcc-m02   Ready    <none>          125m   v1.24.1
```

```
$ kubectl get nodes -o wide 
NAME      STATUS   ROLES           AGE    VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE               KERNEL-VERSION   CONTAINER-RUNTIME
vcc       Ready    control-plane   126m   v1.24.1   192.168.39.49   <none>        Buildroot 2021.02.12   5.10.57          docker://20.10.16
vcc-m02   Ready    <none>          125m   v1.24.1   192.168.39.73   <none>        Buildroot 2021.02.12   5.10.57          docker://20.10.16
```

## Install Metallb

MetalLB is a load-balancer implementation for bare metal Kubernetes clusters, using standard routing protocols.

Kubernetes does not offer an implementation of network load balancers (Services of type LoadBalancer) for bare-metal clusters. The implementations of network load balancers that Kubernetes does ship with are all glue code that calls out to various IaaS platforms (GCP, AWS, Azure…).

If you’re not running on a supported IaaS platform (GCP, AWS, Azure…), LoadBalancers will remain in the “pending” state indefinitely when created.

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
```
diego@thinkpad:~$ kubectl get ns
NAME              STATUS   AGE
default           Active   136m
kube-node-lease   Active   136m
kube-public       Active   136m
kube-system       Active   136m
metallb-system    Active   56m
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



minikube -p vcc addons configure metallb

```