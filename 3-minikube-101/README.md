# Minikube
It deploys minikube cluster using kvm2 driver

## Using Linux KVM (Kernel-based Virtual Machine) driver
Deploy minikube using the KVM driver instead to spin up and provision a centos7 machine. To work with KVM, minikube uses the libvirt virtualization API.

### Requirements
* libvirt v1.3.1 or higher
* qemu-kvm v2.0 or higher

### Installing Prerequisites
* Proper installation of KVM and libvirt
* https://github.com/dcmikki/vagrant_automation#install-qemu-kvm-and-libvirt


Once configured, validate that libvirt reports no errors:

`virt-host-validate`

### Usage
Start a cluster using the kvm2 driver:

```
minikube start --driver=kvm2 --cpus=4 --memory=4400mb --disk-size=10000mb
```

```Example
$ minikube start --driver=kvm2 --cpus=4 --memory=4400mb --disk-size=10000mb
😄  minikube v1.26.0 on Linuxmint 20.3
✨  Using the kvm2 driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🔥  Creating kvm2 VM (CPUs=4, Memory=4400MB, Disk=10000MB) ...
🐳  Preparing Kubernetes v1.24.1 on Docker 20.10.16 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

```

### Minikube and Kubectl Autocompletion
For bash users add the following lines in `~/.bashrc`:

```autocompletion
source <(minikube completion bash)
source <(kubectl completion bash)
```

### Minikube defaults configuration
It possible to pass all the defaults configuration in a config.json file, instead to run all oprtion in de command line.

* location --> `~/.minikube/config/config.json`

```config.json
{
    "cpus": 4,
    "disk-size": "12000mb",
    "driver": "kvm2",
    "insecure-registry": "true",
    "memory": "4400mb"
}
```


## Install Minikube multi node cluster

Install minukube with two nodes from the scratch
```
minikube start --driver=kvm2 --cpus=2 --memory=2400mb --disk-size=15000mb --nodes=2 -p vcc
```

### Minikube check commands

```
$ minikube -p vcc node list
vcc     192.168.39.11
vcc-m02 192.168.39.122                                                                  
```

```
$ minikube -p vcc profile list                                                  
|---------|-----------|---------|----------------|------|---------|---------|-------|--------|
| Profile | VM Driver | Runtime |       IP       | Port | Version | Status  | Nodes | Active |
|---------|-----------|---------|----------------|------|---------|---------|-------|--------|
| vcc     | kvm2      | docker  | 192.168.39.11  | 8443 | v1.24.1 | Running |     2 | *      |
|---------|-----------|---------|----------------|------|---------|---------|-------|--------|
```

```
$ minikube -p vcc status
vcc
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

vcc-m02
type: Worker
host: Running
kubelet: Running

```
