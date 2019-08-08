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
step 5 : Install and configure helm in kubernetes
Prerequisites
You should have the following before getting started with the helm setup.

A running Kubernetes cluster.
The Kubernetes cluster API endpoint should be reachable from the machine you are running helm.
Authenticate the cluster using kubectl and it should have cluster admin permissions.

Installing Helm [Client]
This installation is on the client side. ie, a personal workstation, a Linux VM, etc. You can install helm using a single liner. It will automatically find your OS type and installs helm on it.

Execute the following from your command line.
 ```bash
 bash helm_install.sh
```
Create Tiller Service Account With Cluster Admin Permissions
Tiller is the server component for helm. Tiller will be present in the kubernetes cluster and the helm client talks to it for deploying applications using helm charts.

Helm will be managing your cluster resources. So we need to add necessary permissions to the tiller components which resides in the cluster kube-system namespace.

Here is what we will do,

Create a service account names tiller
Create a ClusterRoleBinding with cluster-admin permissions to the tiller service account.
We will add both service account and clusterRoleBinding in one yaml file.
 
Lets create these resources using kubectl

```bash
kubectl apply -f helm-rbac.yaml
```

Initialize Helm: Deploy Tiller
Next step is to initialize helm. When you initialize helm, a deployment named tiller-deploy will be deployed in the kube-system namespace.

If you want a specific tiller version to be installed, you can specify the tiller image link in the init command using --tiller-image flag. You can find the all tiller docker images in public google GCR registry.

```bash
helm init --service-account=tiller --tiller-image=gcr.io/kubernetes-helm/tiller:v2.14.1   --history-max 300
```

You can check the tiller deployment in the kube-system namespace using kubectl.

```bash
kubectl get deployment tiller-deploy -n kube-system
```

### Step 8 & 9 : Setup prometheus and grafana
I have create one yaml to setup prometheus and grafana on kubernetes.we can run this by using kubectl
```bash
kubectl create -f prometheus_grafana.yaml
namespace/monitoring created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
serviceaccount/prometheus-k8s created
configmap/alertmanager-templates created
configmap/alertmanager created
deployment.extensions/alertmanager created
service/alertmanager created
deployment.extensions/grafana-core created
configmap/grafana-import-dashboards created
job.batch/grafana-import-dashboards created
secret/grafana created
service/grafana created
configmap/prometheus-core created
deployment.extensions/prometheus-core created
deployment.extensions/kube-state-metrics created
serviceaccount/kube-state-metrics created
service/kube-state-metrics created
daemonset.extensions/node-directory-size-metrics created
daemonset.extensions/prometheus-node-exporter created
service/prometheus-node-exporter created
configmap/prometheus-rules created
service/prometheus created
```

this yaml file will configure node-exporter,prometheus and grafana

### Step 10: Setup EFK (ElasticSearch, fluentd, Kibana)
Once you've cloned the repository, create the Namespace using kubectl create with the -f filename flag:
```bash
kubectl create -f namespace_efk.yaml
```
You should see the following output:

Output
```bash
namespace/kube-logging created
```
You can then confirm that the Namespace was successfully created:
```bash
kubectl get namespaces
```
At this point, you should see the new kube-logging Namespace:
```bash
Output
NAME           STATUS    AGE
default        Active    23m
kube-logging   Active    1m
kube-public    Active    23m
kube-system    Active    23m
```
We can now deploy an Elasticsearch cluster into this isolated logging Namespace.

#### Create the service using kubectl:

```bash
kubectl create -f elasticSearch-service.yaml
```
You should see the following output:

Output
```bash
service/elasticsearch created
```
Finally, double-check that the service was successfully created using kubectl get:
```bash
kubectl get services --namespace=kube-logging
```
You should see the following:
```bash
Output
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)             AGE
elasticsearch   ClusterIP   None         <none>        9200/TCP,9300/TCP   26s
```
Now that we've set up our headless service and a stable .elasticsearch.kube-logging.svc.cluster.local domain for our Pods, we can go ahead and create the StatefulSet.

#### Creating the StatefulSet

Now, deploy the StatefulSet using kubectl:
```bash
kubectl create -f elasticSearch-statefulset.yaml
You should see the following output:

Output
statefulset.apps/es-cluster created
```
You can monitor the StatefulSet as it is rolled out using kubectl rollout status:
```bash
kubectl rollout status sts/es-cluster --namespace=kube-logging
You should see the following output as the cluster is rolled out:

Output
Waiting for 3 pods to be ready...
Waiting for 2 pods to be ready...
Waiting for 1 pods to be ready...
partitioned roll out complete: 3 new pods have been updated...
```
Once all the Pods have been deployed, you can check that your Elasticsearch cluster is functioning correctly by performing a request against the REST API.

To do so, first forward the local port 9200 to the port 9200 on one of the Elasticsearch nodes (es-cluster-0) using kubectl port-forward:
```bash
kubectl port-forward es-cluster-0 9200:9200 --namespace=kube-logging
```
Then, in a separate terminal window, perform a curl request against the REST API:
```bash
curl http://localhost:9200/_cluster/state?pretty
```
You shoulds see the following output:
```bash
Output
{
  "cluster_name" : "k8s-logs",
  "compressed_size_in_bytes" : 348,
  "cluster_uuid" : "QD06dK7CQgids-GQZooNVw",
  "version" : 3,
  "state_uuid" : "mjNIWXAzQVuxNNOQ7xR-qg",
  "master_node" : "IdM5B7cUQWqFgIHXBp0JDg",
  "blocks" : { },
  "nodes" : {
    "u7DoTpMmSCixOoictzHItA" : {
      "name" : "es-cluster-1",
      "ephemeral_id" : "ZlBflnXKRMC4RvEACHIVdg",
      "transport_address" : "10.244.8.2:9300",
      "attributes" : { }
    },
    "IdM5B7cUQWqFgIHXBp0JDg" : {
      "name" : "es-cluster-0",
      "ephemeral_id" : "JTk1FDdFQuWbSFAtBxdxAQ",
      "transport_address" : "10.244.44.3:9300",
      "attributes" : { }
    },
    "R8E7xcSUSbGbgrhAdyAKmQ" : {
      "name" : "es-cluster-2",
      "ephemeral_id" : "9wv6ke71Qqy9vk2LgJTqaA",
      "transport_address" : "10.244.40.4:9300",
      "attributes" : { }
    }
  },
...
```
This indicates that our Elasticsearch cluster k8s-logs has successfully been created with 3 nodes: es-cluster-0, es-cluster-1, and es-cluster-2. The current master node is es-cluster-0.

Now that your Elasticsearch cluster is up and running, you can move on to setting up a Kibana frontend for it.

#### Kibana Deployment and Service
This time, we'll create the Service and Deployment in the same file
```bash
kubectl create -f kibana.yaml
You should see the following output:

Output
service/kibana created
deployment.apps/kibana created
```
You can check that the rollout succeeded by running the following command:
```bash
kubectl rollout status deployment/kibana --namespace=kube-logging
You should see the following output:

Output
deployment "kibana" successfully rolled out
```
To access the Kibana interface, we'll once again forward a local port to the Kubernetes node running Kibana. Grab the Kibana Pod details using kubectl get:
```bash
kubectl get pods --namespace=kube-logging
Output
NAME                      READY     STATUS    RESTARTS   AGE
es-cluster-0              1/1       Running   0          55m
es-cluster-1              1/1       Running   0          54m
es-cluster-2              1/1       Running   0          54m
kibana-6c9fb4b5b7-plbg2   1/1       Running   0          4m27s
```
Here we observe that our Kibana Pod is called kibana-6c9fb4b5b7-plbg2.

Forward the local port 5601 to port 5601 on this Pod:
```bash
kubectl port-forward kibana-6c9fb4b5b7-plbg2 5601:5601 --namespace=kube-logging
```
You should see the following output:
```bash
Output
Forwarding from 127.0.0.1:5601 -> 5601
Forwarding from [::1]:5601 -> 5601
```
Now, in your web browser, visit the following URL:
```bash
http://localhost:5601
```

#### Fluentd DaemonSet

Now, roll out the DaemonSet using kubectl:
```bash
kubectl create -f fluentd.yaml
```
You should see the following output:
```bash
Output
serviceaccount/fluentd created
clusterrole.rbac.authorization.k8s.io/fluentd created
clusterrolebinding.rbac.authorization.k8s.io/fluentd created
daemonset.extensions/fluentd created
```
Verify that your DaemonSet rolled out successfully using kubectl:
```bash
kubectl get ds --namespace=kube-logging
```
You should see the following status output:
```bash
Output
NAME      DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd   3         3         3         3            3           <none>          58s
```
This indicates that there are 3 fluentd Pods running, which corresponds to the number of nodes in our Kubernetes cluster.

We can now check Kibana to verify that log data is being properly collected and shipped to Elasticsearch.

With the kubectl port-forward still open, navigate to http://localhost:5601.
