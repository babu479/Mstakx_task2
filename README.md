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
```bash
Output
● jenkins.service - LSB: Start Jenkins at boot time
  Loaded: loaded (/etc/init.d/jenkins; bad; vendor preset: enabled)
  Active:active (exited) since Thu 2017-04-20 16:51:13 UTC; 2min 7s ago
    Docs: man:systemd-sysv-generator(8)
```
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

#### Customize Jenkins Screen

We'll click the "Install suggested plugins" option, which will immediately begin the installation process:

Jenkins Getting Started Install Plugins Screen

When the installation is complete, we'll be prompted to set up the first administrative user. It's possible to skip this step and continue as admin using the initial password we used above, but we'll take a moment to create the user.

Note: The default Jenkins server is NOT encrypted, so the data submitted with this form is not protected. When you're ready to use this installation, follow the guide How to Configure Jenkins with SSL using an Nginx Reverse Proxy. This will protect user credentials and information about builds that are transmitted via the Web interface.

Jenkins Create First Admin User Screen

Once the first admin user is in place, you should see a "Jenkins is ready!" confirmation screen.

Jenkins is ready screen
Click "Start using Jenkins" to visit the main

### Create jenkins pipeline Job for CI/CD

I have written a jenkinsfile which contains multiple stages to create docker image and push that image into the docker repository
#### variables:
```bash
def BLUE_CONTAINER_NAME="blue-container"
def GREEN_CONTAINER_NAME="green-container"
def BLUE_CONTAINER_NAME_TAG="babu479/blue_demo_application"
def GREEN_CONTAINER_NAME_TAG="babu479/green_demo_application"
def CONTAINER_TAG="latest"
def DOCKER_HUB_USER="babu479"
def HTTP_PORT="8080"
```

#### Stage 1: Intialize
In this stage, I am intilizing maven with by using global tool configuration
```bash
stage ('Initialize') {
        def mavenHome  = tool 'maven'
        env.PATH = "${mavenHome}/bin:${env.PATH}"
}
```
#### Stage 2: clone
In this stage, I am cloning my demo application code into two different directories
one of the application with background blue color and antoher with green but both are in different directories
```bash
stage ('Clone') {
dir("blue"){
git branch: 'blue',
credentialsId: 'ae6c9066a-410c-4462-b2ed-56004c6c20ab',
url: 'https://github.com/babu479/Demo-Application.git'
}
dir("green"){
git branch: 'green',
credentialsId: 'ae6c9066a-410c-4462-b2ed-56004c6c20ab',
url: 'https://github.com/babu479/Demo-Application.git'
}
}
```
#### stage 3: Build
In this stage, I am running maven install command to build war to demo applications

```bash
stage ('Build') {
sh """
cd $WORKSPACE/blue
mvn clean install"""
sh """
cd $WORKSPACE/green
mvn clean install"""
}
```
#### stage 4 & 5 : purge the old container and Build new image 
```bash
stage ("Image Prune") {
    imagePrune(BLUE_CONTAINER_NAME, GREEN_CONTAINER_NAME)
}
stage('Image Build'){
        imageBuild(BLUE_CONTAINER_NAME, GREEN_CONTAINER_NAME,CONTAINER_TAG)
    }
	def imagePrune(blueContainerName, greenContainerName){
    try {
        sh """id
        docker image prune -f
        docker stop $blueContainerName
        whoami"""
	sh """id
        docker image prune -f
        docker stop $greenContainerName
        whoami"""
    } catch(error){}
}
def imageBuild(blueContainerName, greenContainerName, tag){
    sh """
    cd $WORKSPACE/blue
    cp $WORKSPACE/blue/target/*.war $WORKSPACE/blue
	docker build -t $blueContainerName:$tag  -t $blueContainerName --pull --no-cache ."""
     sh """
    cd $WORKSPACE/green
    cp $WORKSPACE/green/target/*.war $WORKSPACE/green
	docker build -t $greenContainerName:$tag  -t $greenContainerName --pull --no-cache ."""
    echo "Image build complete"
}
```
#### stage 6: push image to repository
```bash
stage('Push to Docker Registry'){
        withCredentials([usernamePassword(credentialsId: 'DockerHubAccount', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            pushToImage(BLUE_CONTAINER_NAME,GREEN_CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
        }
def pushToImage(blueContainerName,greenContainerName, tag, dockerUser, dockerPassword){
    sh """docker login -u $dockerUser -p $dockerPassword
    docker tag $blueContainerName:$tag $dockerUser/$blueContainerName:$tag
    docker push $dockerUser/$blueContainerName:$tag """
    sh """docker login -u $dockerUser -p $dockerPassword
    docker tag $greenContainerName:$tag $dockerUser/$greenContainerName:$tag
    docker push $dockerUser/$greenContainerName:$tag """
    
    echo "Image push complete"
	}
```
	
	
### step 3 & 4 : Create development namespace and sample application deployment 
I have selected microservices application to deploy in development namespace
because of deploy of microservice is important for kubernetes.

```bash
kubectl create -f complete-demo-of-sock-shop-microservices.yaml
kubectl get pods -n development 
```
if all the pods are running fine, then try to access the application with nodeport of frontend microservice service 
```bash
curl http://<master-sever-Ip>:NodePort/
```

