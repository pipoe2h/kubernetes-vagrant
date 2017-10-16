# -*- mode: ruby -*-
# vi: set ft=ruby :

# Variables section
## Infrastructure
$master_cpu = 2
$master_memory = 2048
$node_count = 2 # Minimum one node
$node_cpu = 2
$node_memory = 2048
$linked_clone = true # Save storage space
$network = "192.168.34" # Only first three octets
$domain = "k8s.local"

## Kubernetes
$k8s_version = "1.8.1"
$k8s_token = "b33f0a.59a7100c41aa5999"
$k8s_api_port = "6443"

## Canal
$canal_rbac_url = "https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.7/rbac.yaml"
$canal_url = "https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.7/canal.yaml"

######### DO NOT MODIFY AFTER THIS LINE #########
## Infrastructure
$box_image = "ubuntu/xenial64"

## Scripts
$build_prereq = <<SCRIPT
echo "...Setting network and memory..."
sysctl net.bridge.bridge-nf-call-iptables=1
swapoff -a

echo "...Installing dependencies..."
apt-get update -y && apt-get upgrade -y && apt-get install -y ebtables ethtool curl apt-transport-https

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

Vagrant.configure("2") do |config|

    config.vm.define "k8s-master" do |master|
        master.vm.box = $box_image
        master.vm.hostname = "master.#{$domain}"
        master.vm.network :private_network, ip: "#{$network}.10"
        master.vm.provider "virtualbox" do |vb|
            vb.memory = $master_memory
            vb.cpus = $master_cpu
            vb.linked_clone = $linked_clone
        end
        master.vm.provision "shell", inline: <<-SHELL
        #{$build_prereq}
        #{$kubeadm_init}
        #{$kubectl_canal}
        #{$kubectl_config}
        SHELL
    end 

    (1..$node_count).each do |i|
        config.vm.define "k8s-node#{i}" do |node|
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