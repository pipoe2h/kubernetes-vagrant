# -*- mode: ruby -*-
# vi: set ft=ruby :

# Variables section
## Infrastructure
$master_cpu = 2
$master_memory = 2048                       # 1GB memory makes the deployment fail    
$node_count = 2                             # Minimum one node
$node_cpu = 2           
$node_memory = 2048                         # 1GB memory makes the deployment fail
$linked_clone = true                        # Save storage space
$network = "192.168.34"                     # Only first three octets
$domain = "k8s.local"

## Kubernetes
$k8s_version = "1.8.1"                      # Find other versions on https://github.com/kubernetes/kubernetes/releases
$k8s_token = "b33f0a.59a7100c41aa5999"      # This is a static token to make possible the automation. You can replace it with your own token 
$k8s_api_port = "6443"                      # This is the default Kubernetes API port when kubeadm is used

## NFS
$nfs_gb = 10                                # The NFS disk for the master server is expressed in decimal gigabytes (Default: 10GB)

######### DO NOT MODIFY AFTER THIS LINE #########
## Infrastructure
$box_image = "ubuntu/xenial64"

## Canal
$canal_rbac_url = "https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.7/rbac.yaml"
$canal_url = "https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.7/canal.yaml"

## Scripts
$build_prereq = <<SCRIPT
echo "...Setting network and memory..."
sysctl net.bridge.bridge-nf-call-iptables=1
swapoff -a

echo "...Installing dependencies..."
apt-get update -y && apt-get upgrade -y && apt-get install -y ebtables ethtool curl apt-transport-https nfs-common

echo "...Installing Docker..."
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/docker.list 
deb https://download.docker.com/linux/$(lsb_release -si | tr '[:upper:]' '[:lower:]') $(lsb_release -cs) stable 
EOF

apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')
usermod -a -G docker ubuntu

echo "...Installing kubeadm..."
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update && apt-get install -y kubelet kubeadm kubectl
SCRIPT

$kubeadm_init = <<SCRIPT
kubeadm init --apiserver-advertise-address=#{$network}.10 --kubernetes-version=#{$k8s_version} --pod-network-cidr=10.244.0.0/16 \
--token=#{$k8s_token}
SCRIPT

$kubectl_canal = <<SCRIPT
kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f #{$canal_rbac_url}
kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f #{$canal_url}
SCRIPT

$kubeadm_join = <<SCRIPT
kubeadm join --token #{$k8s_token} #{$network}.10:#{$k8s_api_port} --discovery-token-unsafe-skip-ca-verification
SCRIPT

$kubectl_config = <<SCRIPT
mkdir /home/ubuntu/.kube
cp /etc/kubernetes/admin.conf /home/ubuntu/.kube/config
chown -R ubuntu:ubuntu /home/ubuntu/.kube
SCRIPT

$build_nfs = <<SCRIPT
parted -s -a optimal /dev/sdc mklabel GPT mkpart primary 0% 100% set 1 lvm on
pvcreate /dev/sdc1
vgcreate kubernetes /dev/sdc1
lvcreate -l 100%FREE -n nfs kubernetes
mkfs.ext4 /dev/mapper/kubernetes-nfs
mkdir -p /var/nfs/kubernetes
echo "/dev/mapper/kubernetes-nfs    /var/nfs/kubernetes ext4    defaults    0   2" >> /etc/fstab
mount -a
apt-get update && apt-get install -y nfs-kernel-server
chown nobody:nogroup /var/nfs/kubernetes
echo "/var/nfs/kubernetes   *(rw,sync,no_subtree_check)" >> /etc/exports
systemctl enable nfs-kernel-server.service && systemctl restart nfs-kernel-server.service
SCRIPT

Vagrant.configure("2") do |config|

    file_root = File.dirname(File.expand_path(__FILE__))

    config.vm.define "master" do |master|
        master.vm.box = $box_image
        master.vm.hostname = "master.#{$domain}"
        master.vm.network :private_network, ip: "#{$network}.10"
        master.vm.provider "virtualbox" do |vb|
            vb.memory = $master_memory
            vb.cpus = $master_cpu
            vb.linked_clone = $linked_clone
            vb.customize ["modifyvm", :id, "--macaddress1", "auto"]
            file_to_disk = File.join(file_root, "nfs.vdi")
            unless File.exist?(file_to_disk)
                vb.customize ['createhd', '--filename', file_to_disk, '--format', 'VDI', '--size', $nfs_gb * 1024]
            end
            vb.customize ['storageattach', :id,  '--storagectl', 'SCSI', '--port', 2, '--type', 'hdd', '--medium', file_to_disk]
        end
        master.vm.provision "shell", inline: <<-SHELL
        #{$build_prereq}
        #{$kubeadm_init}
        #{$kubectl_canal}
        #{$kubectl_config}
        #{$build_nfs}
        SHELL
    end 

    (1..$node_count).each do |i|
        config.vm.define "node#{i}" do |node|
            node.vm.box = $box_image
            node.vm.hostname = "node#{i}.#{$domain}"
            node.vm.network :private_network, ip: "#{$network}.#{i + 10}"
            node.vm.provider "virtualbox" do |vb|
                vb.memory = $node_memory
                vb.cpus = $node_cpu
                vb.linked_clone = $linked_clone
            end
            node.vm.provision "shell", inline: <<-SHELL
            #{$build_prereq}
            #{$kubeadm_join}
            SHELL
        end         
    end
end    
