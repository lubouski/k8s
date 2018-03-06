# -*- mode: ruby -*-
# vi: set ft=ruby :

$basic_setup = <<SCRIPT
yum update
yum install -y docker
cat <<EOF > /etc/docker/daemon.json
{
  "exec-opt": ["native.cgroupdriver=systemd"]
}
EOF
systemctl enable docker && systemctl start docker
setenforce 0
sysctl net.bridge.bridge-nf-call-iptables=1
swapoff -a
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
SCRIPT

$bootstrap_token = "4vaecq.w6vg0yjs3tefxqnm"

$master_setup = <<SCRIPT
kubeadm init --apiserver-advertise-address 192.168.100.2 --pod-network-cidr 10.244.0.0/16 --token #{$bootstrap_token}
kubeadm token delete #{$bootstrap_token}
kubeadm token create #{$bootstrap_token} --description "Bootstrap token used for joining new nodes" --groups system:bootstrappers:kubeadm:default-node-token --ttl 0 --usages signing,authentication
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl cluster-info
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
kubectl get pods --all-namespaces
SCRIPT

$minion_setup = <<SCRIPT
kubeadm join --token #{$bootstrap_token} --discovery-token-unsafe-skip-ca-verification 192.168.100.2:6443
SCRIPT

Vagrant.configure('2') do |config|
  # define box to use
  config.vm.box = 'sbeliakou/centos-7.4-x86_64-minimal'

  # define nodes
  # nodes = { 'master.lab' => '10.0.0.2', 'node1.lab' => '10.0.0.3', 'node2.lab' => '10.0.0.4', 'node3.lab' => '10.0.0.5' }
  nodes = { 'master.lab' => '192.168.100.2', 'node1.lab' => '192.168.100.3', 'node2.lab' => '192.168.100.4' }

  # make hosts file
  hosts = []
  nodes.each{ |key, value| hosts.push("#{value} #{key}") }

  # provision nodes
  nodes.each_key do |node|
    config.vm.define node do |item|
      item.vm.hostname = node
      item.vm.network 'private_network', ip: nodes[node]

      item.vm.provider 'virtualbox' do |v|
        v.name = node
	      v.memory = 2048
        v.linked_clone = true
      end
      item.vm.provision 'shell', inline: $basic_setup, name: 'basic setup'
      if node.include? 'master' then
        item.vm.provision 'shell', inline: $master_setup, name: 'master setup'
      else
        item.vm.provision 'shell', inline: $minion_setup, name: 'minion setup'
      end
    end
  end
end

### Сluster init
# cat /dev/urandom | tr -dc 'a-z0-9*23' | fold -w 23 | head -n 1 | sed s/././7
# kubeadm init --apiserver-advertise-address=192.168.100.2 --pod-network-cidr=10.0.0.0/16
# mkdir -p $HOME/.kube
# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown $(id -u):$(id -g) $HOME/.kube/config

### Deleting node/cluster
# kubectl drain master.lab --delete-local-data --force --ignore-daemonsets
# kubectl delete node master.lab
# kubectl reset

### Сlient join
# kubeadm join --discovery-token abcdef.1234567890abcdef --discovery-token-unsafe-skip-ca-verification 192.168.100.2:6443
#

### Bootstrap token
# kubeadm token create 4vaecq.w6vg0yjs3tefxqnm --description "Bootstrap token used for new nodes" --groups system:bootstrappers:kubeadm:default-node-token --ttl 0 --usages signing,authentication
