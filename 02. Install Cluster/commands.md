### Provision Infrastructure 

##### move private key to .ssh folder and restrict access
    mv ~/Downloads/k8s-node.pem ~/.ssh
    chmod 400 ~/.ssh/k8s-node.pem

##### ssh into ec2 instance with its public ip
    ssh -i ~/.ssh/k8s-node.pem ubuntu@35.180.130.108


### Configure Infrastructure
    sudo swapoff -a

##### set host names of nodes
    sudo vim /etc/hosts

##### get private ips of each node and add this to each server 
    172.31.48.251 controlplane
    172.31.48.18 worker1
    172.31.46.180 worker2

##### we can now use these names instead of typing the IPs, when nodes talk to each other. After that, assign a hostname to each of these servers.

##### on master server
    sudo hostnamectl set-hostname controlplane 

##### on worker1 server
    sudo hostnamectl set-hostname worker1 

##### on worker2 server
    sudo hostnamectl set-hostname worker2


### Initialize K8s cluster
    sudo kubeadm init

### Check kubelet process running 
    service kubelet status
    systemctl status kubelet

### Check extended logs of kubelet service
    journalctl -u kubelet

### Access cluster as admin
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

### Kubectl commands

##### get node information
    kubectl get node

##### get pods in kube-system namespace
    kubectl get pod -n kube-system

##### get pods from all namespaces
    kubectl get pod -A

##### get wide output
    kubectl get pod -n kube-system -o wide


### Install pod network plugin

##### Install CLI 

    CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
    CLI_ARCH=amd64
    if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
    curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
    sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
    sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
    rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
    
##### Install the plugin 
    cilium install 
[Link to the Cilium installation guide](https://docs.cilium.io/en/latest/gettingstarted/k8s-install-default/)    

##### check cilium status
    cilium status
    kubectl -n kube-system exec cilium-2hq5z -- cilium-dbg status
    cilium connectivity test

### Join worker nodes

##### create and execute script
    vim install-containerd.sh
    chmod u+x install-containerd.sh
    ./install-containerd.sh

##### on control plane
    kubeadm token create --help
    kubeadm token create --print-join-command

##### copy the output command and execute on worker node as ROOT
    sudo kubeadm join 172.31.43.99:6443 --token 9bds1l.3g9ypte9gf69b5ft --discovery-token-ca-cert-hash sha256:xxxx

##### start a test pod
    kubectl run test --image=nginx


