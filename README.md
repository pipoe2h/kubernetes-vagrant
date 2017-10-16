# kubernetes-vagrant
This Vagrantfile is for the deployment of a Kubernetes platform with a single master and multiple nodes. This platform uses Canal as the CNI plug-in.

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
  * Single Kubernetes master server (2x CPU/2GB memory)
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

Usage
-----

    $ vagrant up
    
Depending on your hardware performance and Internet speed, a single Kubernetes node platform (master is always required) can take around 5-10 minutes to come up.

You can check the Kubernetes cluster status on master server.

    $ vagrant ssh k8s-master
    [ubuntu@k8s-master:~ ] $ kubectl get nodes
