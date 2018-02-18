# kubernetes-vagrant
This Vagrantfile is for the deployment of a Kubernetes platform with a single master and multiple nodes. This platform uses Canal as the CNI plug-in.

In addition, a NFS server provides persistent storage for your pods. The path is <IP_nfs_server>:/var/nfs/kubernetes

Pre-requisites
--------------
* VirtualBox

Installation
------------

    $ git clone https://github.com/pipoe2h/kubernetes-vagrant.git
    $ cd kubernetes-vagrant

Configuration
-------------
The Kubernetes platform you get with this Vagrantfile is:
  * NFS server (1x CPU/256MB memory/10GB disk/IP <network>.9)
  * Single Kubernetes master server (1x CPU/1GB memory/IP <network>.10)
  * Two Kubernetes node servers (1x CPU/1GB memory/IP <network>.11-254)
  * Vagrant box Ubuntu/Xenial64
  
Before you run `vagrant up` you should review the Vagrantfile settings to map your requirements.

```ruby
## Infrastructure
### General
$linked_clone = true                        # Save storage space
$network = "192.168.34"                     # Only first three octets
$vagrant_user = "vagrant"                   # The SSH user included in the vagrant box

### NFS
$nfs_cpu = 1
$nfs_memory = 256
$nfs_gb = 10                                # The NFS disk for the master server is expressed in decimal gigabytes (Default: 10GB)

### Master
$master_cpu = 1
$master_memory = 1024                       # 1GB minimum required (2GB recommended)    

### Node
$node_count = 2                             # Minimum one node
$node_cpu = 1           
$node_memory = 1024                         # 1GB minimum required (2GB recommended)

## Docker & Kubernetes
$docker_version = "17.03"                   # Find other versions on https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-docker
$k8s_token = "b33f0a.59a7100c41aa5999"      # This is a static token to make possible the automation. You can replace it with your own token 
$k8s_api_port = "6443"                      # This is the default Kubernetes API port when kubeadm is used
```

Usage
-----

    $ vagrant up
    
Depending on your hardware performance and Internet speed, a single Kubernetes node platform (master is always required) can take around 5-10 minutes to come up.

You can check the Kubernetes cluster status on master server.

    $ vagrant ssh master

    vagrant@master:~$ kubectl get nodes
    NAME      STATUS    ROLES     AGE       VERSION
    master    Ready     master    14h       v1.9.3
    node1     Ready     <none>    14h       v1.9.3
    node2     Ready     <none>    14h       v1.9.3

