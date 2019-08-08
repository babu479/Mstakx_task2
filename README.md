# Mstakx_task2

## TASK2

Login to AWS dashboard and  create an instance on AWS. For this tutorial, will be using t2.medium instance. Make sure that the security group allows ssh, port 80, 8080 and 6443. Also, open all traffic between the subnet thus:

All traffic
All
All
172.31.0.0/16
Click here on how to create an instance on AWS.

Step 2:
ssh into the instance and install the base image using the following steps

ubuntu@ip-172-31-49-128:~$ sudo apt-get update

ubuntu@ip-172-31-49-128:~$ sudo apt-get install -y apt-transport-https

ubuntu@ip-172-31-49-128:~$ sudo su -
 root@ip-172-31-49-128:~# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
 root@ip-172-31-49-128:~# cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
 > deb http://apt.kubernetes.io/ kubernetes-xenial main
 > EOF

root@ip-172-31-49-128:~# apt-get update
Step 3: Install Docker
root@ip-172-31-49-128:~# apt-get install -y docker.io
Step 4: Install kubeadm, Kubelet, Kubectl and Kubernetes-cni
root@ip-172-31-49-128:~# apt-get install -y kubelet kubeadm kubectl kubernetes-cni
Note: remember to create an AMI image at this point.
Section 2: Creating the Kubernetes Master Node
Lets initialize the cluster master node using kubeadm init

root@ip-172-31-49-128:~# kubeadm init
After this command finished running, the following message will be thrown on the screen: Remember to read and follow the instructions.

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
 https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

kubeadm join --token 844a02.ed299ddcbe17430a 172.31.49.128:6443 --discovery-token-ca-cert-hash sha256:17463c630785dd8685fdd7531389382ce302644db6c329197e20e271aab0bf32
Lets test our cluster

ubuntu@ip-172-31-49-128:~$ kubectl get node
 The connection to the server localhost:8080 was refused - did you specify the right host or port?
We had this error because we have not followed the instructions shown above. You must first run  the following as a non-root user:

 ubuntu@ip-172-31-49-128:~$mkdir -p $HOME/.kube
 ubuntu@ip-172-31-49-128:~$sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 ubuntu@ip-172-31-49-128:~$sudo chown $(id -u):$(id -g) $HOME/.kube/config
ubuntu@ip-172-31-49-128:~$ kubectl get node
 NAME STATUS ROLES AGE VERSION
 ip-172-31-49-128 NotReady master 5m v1.9.2
Now  have been able to run the kubectl command and its showing  one node running. Note that the master node is showing NotReady status. We want the master to show Ready. The reason for this is because kube-dns (kube-system kube-dns-6f4fd4bdf-2q7z2) is pending as shown below. The NotReady status will disappear after a CNI network is installed.

ubuntu@ip-172-31-49-128:~$ kubectl get pods --all-namespaces
NAMESPACE NAME READY STATUS RESTARTS AGE
kube-system etcd-ip-172-31-49-128 1/1 Running 0 9m
kube-system kube-apiserver-ip-172-31-49-128 1/1 Running 0 9m
kube-system kube-controller-manager-ip-172-31-49-128 1/1 Running 0 9m
kube-system kube-dns-6f4fd4bdf-n28st 0/3 Pending 0 10m
kube-system kube-proxy-kpb77 1/1 Running 0 10m
kube-system kube-scheduler-ip-172-31-49-128 1/1 Running 0 9m
Section 3: Installing a CNI Network.
A network is needed to enable the pods to communicate with each other. WEAVE CNI plugin is the network plugin used here.

root@ip-172-31-49-128:~# sysctl net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-iptables = 1

ubuntu@ip-172-31-49-128:~$ export kubever=$(kubectl version | base64 | tr -d '\n')
ubuntu@ip-172-31-49-128:~$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
serviceaccount "weave-net" created
clusterrole "weave-net" created
clusterrolebinding "weave-net" created
role "weave-net" created
rolebinding "weave-net" created
daemonset "weave-net" created

Now that the CNI network has been created, give it a minute or 2 and  test again. The result is as shown

ubuntu@ip-172-31-49-128:~$ kubectl get nodes
NAME STATUS ROLES AGE VERSION
ip-172-31-49-128 Ready master 10m v1.9.2

ubuntu@ip-172-31-49-128:~$ kubectl get pods --all-namespaces
NAMESPACE NAME READY STATUS RESTARTS AGE
kube-system etcd-ip-172-31-49-128 1/1 Running 0 9m
kube-system kube-apiserver-ip-172-31-49-128 1/1 Running 0 9m
kube-system kube-controller-manager-ip-172-31-49-128 1/1 Running 0 9m
kube-system kube-dns-6f4fd4bdf-n28st 3/3 Running 0 10m
kube-system kube-proxy-kpb77 1/1 Running 0 10m
kube-system kube-scheduler-ip-172-31-49-128 1/1 Running 0 9m
kube-system weave-net-n4sdm 2/2 Running 0 2m
Here the CNI weave network has been installed and as a result the master node is now showing ‘READY’ and kube-dns is now showing ‘Running’ instead of pending. You will also notice the creation of a weave container weave-net-n4sdm listed above.

Section 4: Creating the Kubernetes Slave Nodes
Its time to install the Kubernetes Slave nodes.

Create an instance using the AMI that was created above. For test purposes, its OK to choose t2. micro image.

Step 1: Join the first Node to the Cluster. ssh into the slave node
Step 2: run the join command from kubeadm init screen output above
ubuntu@ip-172-31-63-83:~$ sudo su -
 root@ip-172-31-63-83:~# kubeadm join --token 844a02.ed299ddcbe17430a 172.31.49.128:6443 --discovery-token-ca-cert-hash sha256:17463c630785dd8685fdd7531389382ce302644db6c329197e20e271aab0bf32
 [preflight] Running pre-flight checks.
 [WARNING FileExisting-crictl]: crictl not found in system path
 [discovery] Trying to connect to API Server "172.31.49.128:6443"
 [discovery] Created cluster-info discovery client, requesting info from "https://172.31.49.128:6443"
 [discovery] Requesting info from "https://172.31.49.128:6443" again to validate TLS against the pinned public key
 [discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "172.31.49.128:6443"
 [discovery] Successfully established connection with API Server "172.31.49.128:6443"

This node has joined the cluster:
 * Certificate signing request was sent to master and a response was received.
 * The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.

Step 3: Check that the nodes are ready.
ubuntu@ip-172-31-49-128:~$ kubectl get nodes
 NAME STATUS ROLES AGE VERSION
 ip-172-31-49-128 Ready master 1h v1.9.2
 ip-172-31-63-83 Ready <none> 1m v1.9.2
Step 4: Check that the pods are all up. Note how kube-proxy and weave-net gets added with every new node.
ubuntu@ip-172-31-49-128:~$ kubectl get pods --all-namespaces
 NAMESPACE NAME READY STATUS RESTARTS AGE
 kube-system etcd-ip-172-31-49-128 1/1 Running 0 1h
 kube-system kube-apiserver-ip-172-31-49-128 1/1 Running 0 1h
 kube-system kube-controller-manager-ip-172-31-49-128 1/1 Running 0 1h
 kube-system kube-dns-6f4fd4bdf-n28st 3/3 Running 0 1h
 kube-system kube-proxy-kpb77 1/1 Running 0 1h
 kube-system kube-proxy-wk4lv 1/1 Running 0 2m
 kube-system kube-scheduler-ip-172-31-49-128 1/1 Running 0 1h
 kube-system weave-net-kkzc6 2/2 Running 0 2m
 kube-system weave-net-n4sdm 2/2 Running 0 1h
Step 5: Join the second  node to the Cluster. ssh into the slave node.
ubuntu@ip-172-31-59-129:~$ sudo su -
 root@ip-172-31-59-129:~# kubeadm join --token 844a02.ed299ddcbe17430a 172.31.49.128:6443 --discovery-token-ca-cert-hash sha256:17463c630785dd8685fdd7531389382ce302644db6c329197e20e271aab0bf32
 [preflight] Running pre-flight checks.
 [WARNING FileExisting-crictl]: crictl not found in system path
 [discovery] Trying to connect to API Server "172.31.49.128:6443"
 [discovery] Created cluster-info discovery client, requesting info from "https://172.31.49.128:6443"
 [discovery] Requesting info from "https://172.31.49.128:6443" again to validate TLS against the pinned public key
 [discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "172.31.49.128:6443"
 [discovery] Successfully established connection with API Server "172.31.49.128:6443"

This node has joined the cluster:
 * Certificate signing request was sent to master and a response was received.
 * The Kubelet was informed of the new secure connection details.
Step 6: Check that the second node has joined the cluster and ready.
Run 'kubectl get nodes' on the master to see this node join the cluster.
ubuntu@ip-172-31-49-128:~$ kubectl get nodes
 NAME STATUS ROLES AGE VERSION
 ip-172-31-49-128 Ready master 2h v1.9.2
 ip-172-31-59-129 Ready <none> 1m v1.9.2
 ip-172-31-63-83 Ready <none> 6m v1.9.2
Step 7: Check that the pods are all up. Note how kube-proxy and weave-net gets added with every new node.
ubuntu@ip-172-31-49-128:~$ kubectl get pods --all-namespaces
 NAMESPACE NAME READY STATUS RESTARTS AGE
 kube-system etcd-ip-172-31-49-128 1/1 Running 0 1h
 kube-system kube-apiserver-ip-172-31-49-128 1/1 Running 0 1h
 kube-system kube-controller-manager-ip-172-31-49-128 1/1 Running 0 2h
 kube-system kube-dns-6f4fd4bdf-n28st 3/3 Running 0 2h
 kube-system kube-proxy-kpb77 1/1 Running 0 2h
 kube-system kube-proxy-smgtc 1/1 Running 0 46s
 kube-system kube-proxy-wk4lv 1/1 Running 0 6m
 kube-system kube-scheduler-ip-172-31-49-128 1/1 Running 0 1h
 kube-system weave-net-9z8fx 2/2 Running 0 46s
 kube-system weave-net-kkzc6 2/2 Running 0 6m
 kube-system weave-net-n4sdm 2/2 Running 0 1h
