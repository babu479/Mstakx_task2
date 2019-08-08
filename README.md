# Mstakx_task2

## TASK2

### Step 1: Created Kubernetes cluster using kubeadm
#### ssh into the instance and install the base image using the following steps
sudo apt-get update
sudo apt-get install -y apt-transport-https
sudo su -
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
> deb http://apt.kubernetes.io/ kubernetes-xenial main
> EOF
#### Install Docker
apt-get update
apt-get install -y docker.io
#### Install kubeadm, Kubelet, Kubectl and Kubernetes-cni
apt-get install -y kubelet kubeadm kubectl kubernetes-cni
#### Creating the Kubernetes Master Node
 Lets initialize the cluster master node using kubeadm init
 kubeadm init
 After this command finished running, the following message will be thrown on the screen: Remember to read and follow the instructions.
 kubectl get node
 We had this error because we have not followed the instructions shown above,you must first run  the following as a non-root user
 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
 kubectl get node
 kubectl get pods --all-namespaces
 ##### Now  have been able to run the kubectl command and its showing  one node running. Note that the master node is showing NotReady status. We want the master to show Ready. The reason for this is because kube-dns (kube-system kube-dns-6f4fd4bdf-2q7z2) is pending as shown below. The NotReady status will disappear after a CNI network is installed.

Installing a CNI Network.
sudo su -
sysctl net.bridge.bridge-nf-call-iptables=1
export kubever=$(kubectl version | base64 | tr -d '\n')
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
exit
kubectl get nodes

install worker nodes, the first node
 sudo su -
 kubeadm join 172.31.42.42:6443 --token xueazx.lhzrhmar91wr60ph     --discovery-token-ca-cert-hash sha256:2ca2e1a17c34f8161f90b8ff53e50d38a089a96714b718348e97d51ac9f02b2f

kubectl get nodes
 
 kubectl get pods --all-namespaces
 
 Join the second node to the Cluster. ssh into the slave node.
 
 sudo su -
 kubeadm join 172.31.42.42:6443 --token xueazx.lhzrhmar91wr60ph     --discovery-token-ca-cert-hash sha256:2ca2e1a17c34f8161f90b8ff53e50d38a089a96714b718348e97d51ac9f02b2f
 kubectl get nodes
