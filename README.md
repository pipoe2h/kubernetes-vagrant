# kubernetes-vagrant
This Vagrantfile is for the deployment of a Kubernetes platform with a single master and multiple nodes. This platform uses Canal as the CNI plug-in.

In addition, the master server runs the NFS service to provide persistent storage for your pods. The path is <IP_master_server>:/var/nfs/kubernetes

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
  * Single Kubernetes master server (2x CPU/2GB memory/10GB NFS disk)
  * Two Kubernetes node servers (2x CPU/2GB memory)
  * Vagrant box Ubuntu/Xenial64
  
Before you run `vagrant up` you should review the Vagrantfile settings to map your requirements.

### Number of Kubernetes node servers
```ruby
$node_count = 2                             # Minimum one node
```

### Infrastructure
```ruby
$master_cpu = 2
$master_memory = 2048                       # 1GB memory makes the deployment fail    
$node_cpu = 2           
$node_memory = 2048                         # 1GB memory makes the deployment fail
$linked_clone = true                        # Save storage space
```

### Network and domain
```ruby
$network = "192.168.34"                     # Only first three octets
$domain = "k8s.local"
```

### NFS service
```ruby
$nfs_gb = 10                                # The NFS disk for the master server is expressed in decimal gigabytes (Default: 10GB)
```

Usage
-----

    $ vagrant up
    
Depending on your hardware performance and Internet speed, a single Kubernetes node platform (master is always required) can take around 5-10 minutes to come up.

You can check the Kubernetes cluster status on master server.

    $ vagrant ssh master

    ubuntu@master:~$ kubectl get nodes
    NAME      STATUS    ROLES     AGE       VERSION
    master    Ready     master    14h       v1.8.1
    node1     Ready     <none>    14h       v1.8.1
    node2     Ready     <none>    14h       v1.8.1

