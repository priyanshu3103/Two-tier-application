## Deploy-application-using-GitOps

## About

It seems like you're referring to "GitOps" pipelines. GitOps is a methodology for managing infrastructure and applications for cloud-native environments. It leverages Git as a single source of truth for declarative infrastructure and application code, and relies on automation to manage and update the infrastructure based on changes to Git repositories.
# Prerequisites

- Familiarity with Kubernetes concepts (Pods, Deployments, Services).
- Basic understanding of Docker and containerization.
- Experience with Git for version control.
- Access to a Kubernetes cluster for testing (you can use Minikube, kind, or a cloud provider's Kubernetes service).
- Argo CD and Argo Rollouts documentation: Familiarize yourself with the basics of these tools.


## Steps 
Step 1: Install docker on linux
```bash
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```
Step 2: Install kubectl for linux
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubectl
kubectl version --client
```
Step 3: Install minikube for linux
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb

# Start minikube cluster
  minikube start --nodes 3

# For monitoring the minikube cluster.
  minikube dashboard
``` 
Step 4: Build docker image and pushed on docker hub
```bash
    # Image is already pushed

    docker pull priyanshu1979/flask_app:latest # use this commsnd to pull flaskapp image.
    docker pull mysql:latest # for mysql image
```
Step 5: Setup GitOps pipeline using ArgoCD
```bash
# Install arsocd:
 kubectl create namespace argocd
 kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Change the argocd-server service type to NodePort:
   kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

# Run the argodc-server:
    minikube service argocd-server -n argocd

# login server 
1. user: "admin"
2. For password use this command
   I) kubectl edit secret argocd-initial-admin-secret -n argocd
   II) Copy the password and decode it using this: 'echo "enterpassword" | bash64 --decode'
```


    
## Creating Apps Via UI

1) Open a browser to the Argo CD external UI, and login by visiting the IP/hostname in a browser and use the credentials set in step 4.

After logging in, click the + New App button as shown below:

<img src = "Architecture-3-tier.png">

2) Give your app the name guestbook, use the project default, and leave the sync policy as Manual:

<img src = "Architecture-3-tier.png">

3) Connect the https://github.com/priyanshu3103/Deploy-application-using-GitOps.gitrepo to Argo CD by setting repository url to the github repo url, leave revision as HEAD, and set the path to guestbook and For Destination, set cluster URL to https://kubernetes.default.svc (or in-cluster for cluster name) and namespace to default:

<img src = "Architecture-3-tier.png">

4) Then click on create for last click on "Sync" and the synchronized.




## Setup Argo Rollout
Controller Installation:
```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```
Kubectl Plugin Installation:
```bash
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64
sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
kubectl argo rollouts version
```
1. Deploying a Rollout

Run the following command to deploy the initial Rollout and Service:
```bash
kubectl apply -f mysql-pv-rollout.yml
kubectl apply -f mysql-pvc-rollout.yml
kubectl apply -f mysql-service.yml
kubectl apply -f mysql-rollout.yml
kubectl apply -f configmap.yml
kubectl apply -f 2-tier-service.yml
kubectl apply -f 2-tier-rollout.yml
```
The Argo Rollouts kubectl plugin allows you to visualize the Rollout, its related resources (ReplicaSets, Pods, AnalysisRuns), and presents live state changes as they occur. To watch the rollout as it deploys, run the get rollout --watch command from plugin:

```bash
# For flask rollout:
kubectl argo rollouts get rollout my-rollout --watch
# For mysql rollout:
kubectl argo rollouts get rollout mysql --watch
```
2. Updating a Rollout

Updating the container image :
```bash
kubectl argo rollouts set image my-rollout \
  my-flaskapp=priyanshu1979/flask_app:01

kubectl argo rollouts set image mysql \
  mysql=mysql:5.7
```
3. Monitoring the argo rollouts:
```bash
kubectl argo rollouts dashboard
```

## Clean Up
```bash
kubectl delete deploy mysql
kubectl delete deploy two-tier-app
```
## Approach
- Understanding the Concepts: I started by understanding the core concepts of Argo CD and Argo Rollouts, including their roles in Kubernetes deployments and progressive delivery strategies.
- Research and Documentation: I gathered information from official documentation, community resources, and practical experience to ensure accuracy and completeness in the provided information.
- Step-by-Step Guidance: I structured the information in a step-by-step manner, covering installation, configuration, manifest examples, troubleshooting, and integration with GitOps pipelines.

## Challenges:

- Complexity of Concepts: The concepts of continuous delivery and progressive deployment can be complex, especially for beginners.
- Simplifying these concepts without oversimplifying was a challenge.
- Keeping Information Accurate and Up-to-Date: With the rapidly evolving nature of Kubernetes and related tools, ensuring that the information provided was accurate and up-to-date posed a challenge.

## Solutions:

- Clear and Concise Explanations: To address the complexity of concepts, I focused on providing clear and concise explanations, avoiding technical jargon where possible, and using simple language to convey complex ideas.
- Referencing Official Documentation: To ensure accuracy and currency of information, I heavily relied on official documentation and reputable sources, providing references where necessary.
- Practical Examples and Use Cases: I supplemented theoretical explanations with practical examples and use cases to illustrate concepts and demonstrate their real-world application, making it easier for users to understand and apply the information.