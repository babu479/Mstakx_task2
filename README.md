# Mstakx_task2

## TASK2

### Step 1: Created Kubernetes cluster using kubeadm
#### ssh into the instance and install the base image using the following steps
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https
sudo su -
 curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
 cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
 > deb http://apt.kubernetes.io/ kubernetes-xenial main
 > EOF
 ```
 #### Install Docker
 ```bash
 apt-get update
 apt-get install -y docker.io
 ```
 #### Install kubeadm, Kubelet, Kubectl and Kubernetes-cni
  ```bash
  apt-get install -y kubelet kubeadm kubectl kubernetes-cni
  ```
#### Creating the Kubernetes Master Node
Lets initialize the cluster master node using kubeadm init
 ```bash
 kubeadm init
```

After this command finished running, the following message will be thrown on the screen: Remember to read and follow the instructions.
 ```bash
 kubectl get node
 ```
We had this error because we have not followed the instructions shown above,you must first run  the following as a non-root user
 ```bash
 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config
 ```
 
 ```bash
 kubectl get node
 kubectl get pods --all-namespaces
 ```
Now  have been able to run the kubectl command and its showing  one node running. Note that the master node is showing NotReady status. We want the master to show Ready. The reason for this is because kube-dns (kube-system kube-dns-6f4fd4bdf-2q7z2) is pending as shown below. The NotReady status will disappear after a CNI network is installed.

#### Installing a CNI Network.
 ```bash
 sudo su -
sysctl net.bridge.bridge-nf-call-iptables=1
export kubever=$(kubectl version | base64 | tr -d '\n')
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
exit
kubectl get nodes
```

#### install worker nodes, the first node
  ```bash
  sudo su -
 kubeadm join 172.31.42.42:6443 --token xueazx.lhzrhmar91wr60ph     --discovery-token-ca-cert-hash sha256:2ca2e1a17c34f8161f90b8ff53e50d38a089a96714b718348e97d51ac9f02b2f
kubectl get nodes
kubectl get pods --all-namespaces
```
 
#### Join the second node to the Cluster. ssh into the slave node.
 
  ```bash
  sudo su -
 kubeadm join 172.31.42.42:6443 --token xueazx.lhzrhmar91wr60ph     --discovery-token-ca-cert-hash sha256:2ca2e1a17c34f8161f90b8ff53e50d38a089a96714b718348e97d51ac9f02b2f
 kubectl get nodes
```
### Step 2: CI/CD pipeline using Jenkins outside of the cluster
#### First, we'll add the repository key to the system.

```bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
```
When the key is added, the system will return OK. Next, we'll append the Debian package repository address to the server's sources.list:
```bash
echo deb https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
```
When both of these are in place, we'll run update so that apt-get will use the new repository:
```bash
sudo apt-get update
```
Finally, we'll install Jenkins and its dependencies, including Java:
```bash
sudo apt-get install jenkins
```
Now that Jenkins and its dependencies are in place, we'll start the Jenkins server.
#### Starting Jenkins
Using systemctl we'll start Jenkins:

```bash
sudo systemctl start jenkins
```
Since systemctl doesn't display output, we'll use its status command to verify that it started successfully:
```bash
sudo systemctl status jenkins
```
If everything went well, the beginning of the output should show that the service is active and configured to start at boot:

Output
● jenkins.service - LSB: Start Jenkins at boot time
  Loaded: loaded (/etc/init.d/jenkins; bad; vendor preset: enabled)
  Active:active (exited) since Thu 2017-04-20 16:51:13 UTC; 2min 7s ago
    Docs: man:systemd-sysv-generator(8)
Now that Jenkins is running, we'll adjust our firewall rules so that we can reach Jenkins from a web browser to complete the initial set up.

#### Setting up Jenkins
To set up our installation, we'll visit Jenkins on its default port, 8080, using the server domain name or IP address: http://ip_address_or_domain_name:8080

We should see "Unlock Jenkins" screen, which displays the location of the initial password

Unlock Jenkins screen

In the terminal window, we'll use the cat command to display the password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
We'll copy the 32-character alphanumeric password from the terminal and paste it into the "Administrator password" field, then click "Continue". The next screen presents the option of installing suggested plugins or selecting specific plugins.

Customize Jenkins Screen

We'll click the "Install suggested plugins" option, which will immediately begin the installation process:

Jenkins Getting Started Install Plugins Screen

When the installation is complete, we'll be prompted to set up the first administrative user. It's possible to skip this step and continue as admin using the initial password we used above, but we'll take a moment to create the user.

Note: The default Jenkins server is NOT encrypted, so the data submitted with this form is not protected. When you're ready to use this installation, follow the guide How to Configure Jenkins with SSL using an Nginx Reverse Proxy. This will protect user credentials and information about builds that are transmitted via the Web interface.

Jenkins Create First Admin User Screen

Once the first admin user is in place, you should see a "Jenkins is ready!" confirmation screen.

Jenkins is ready screen
Click "Start using Jenkins" to visit the main
