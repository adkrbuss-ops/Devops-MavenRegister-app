# 🚀 DevOps Project with Jenkins, Maven, SonarQube, Docker, EKS & ArgoCD

A complete end-to-end DevOps implementation demonstrating Continuous Integration, Continuous Delivery, Code Quality Analysis, Containerization, Kubernetes Orchestration, and GitOps deployment practices using industry-standard tools.

## 📌 Project Overview

This project showcases a modern DevOps pipeline that automates the complete software delivery lifecycle:

* Source Code Management with Git & GitHub
* Continuous Integration using Jenkins
* Build Automation using Maven
* Static Code Analysis using SonarQube
* Containerization using Docker
* Container Registry Integration
* Kubernetes Cluster Provisioning using EKS
* GitOps Deployment using ArgoCD
* Infrastructure Automation on AWS

---

## 🏗️ Architecture

![DevOps Architecture](docs/images/devops-architecture.png)

### Architecture Flow

1. Developer pushes code to GitHub.
2. Jenkins Pipeline gets triggered.
3. Maven builds the application.
4. SonarQube performs code quality checks.
5. Docker image is built and pushed to Docker Registry.
6. ArgoCD monitors Kubernetes manifests repository.
7. ArgoCD syncs changes to AWS EKS cluster.
8. Application gets deployed automatically.

---

## 🛠️ Technology Stack

| Category                | Tools            |
| ----------------------- | ---------------- |
| CI/CD                   | Jenkins          |
| Build Tool              | Maven            |
| Code Quality            | SonarQube        |
| Containerization        | Docker           |
| Cloud Platform          | AWS              |
| Container Orchestration | Kubernetes (EKS) |
| GitOps                  | ArgoCD           |
| Database                | PostgreSQL       |
| SCM                     | GitHub           |
| Operating System        | Ubuntu Linux     |

---

# 📋 Prerequisites

Before starting, ensure you have:

* AWS Account
* Ubuntu EC2 Instances
* GitHub Repository
* DockerHub Account
* IAM User with EKS Permissions
* Java 17 Installed
* Internet Connectivity

---

# ⚙️ Jenkins Setup

## Install Java

```bash
sudo apt update
sudo apt upgrade
sudo apt install openjdk-17-jre
java -version
```

## Install Jenkins

Official Documentation:

https://www.jenkins.io/doc/book/installing/linux/

```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins
```

### Start Jenkins

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

### Configure SSH Access

```bash
sudo nano /etc/ssh/sshd_config
sudo service sshd reload

ssh-keygen
# OR
ssh-keygen -t ed25519
```

---

# SonarQube Setup with Docker on AWS EC2 (Ubuntu)

## Prerequisites

* Ubuntu EC2 Instance (Recommended: t3.large or higher)
* Security Group allowing:

  * SSH (22)
  * SonarQube (9000)

---

## Step 1: Update the Server

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 2: Install Docker

Install Docker:

```bash
sudo apt install docker.io -y
```

Enable and start the Docker service:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Add the current user to the Docker group:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Verify Docker installation:

```bash
docker --version
```

Expected Output:

```bash
Docker version XX.X.X, build XXXXXXX
```

---

## Step 3: Configure Linux Kernel Settings

SonarQube uses Elasticsearch internally, which requires specific kernel parameters.

Check current values:

```bash
sysctl vm.max_map_count
sysctl fs.file-max
ulimit -n
ulimit -u
```

Update the required settings:

```bash
sudo sysctl -w vm.max_map_count=524288
sudo sysctl -w fs.file-max=131072
```

Make the changes persistent across reboots:

```bash
echo "vm.max_map_count=524288" | sudo tee -a /etc/sysctl.conf

echo "fs.file-max=131072" | sudo tee -a /etc/sysctl.conf

sudo sysctl -p
```

Verify:

```bash
sysctl vm.max_map_count
sysctl fs.file-max
```

---

## Step 4: Pull and Run SonarQube Container

Pull the latest SonarQube Community Edition image:

```bash
docker pull sonarqube:community
```

Start the container:

```bash
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  sonarqube:community
```

Verify that the container is running:

```bash
docker ps
```

---

## Step 5: Monitor SonarQube Startup Logs

Check container logs:

```bash
docker logs -f sonarqube
```

Wait until the following message appears:

```text
SonarQube is operational
```

This may take a few minutes during the first startup.

---

## Step 6: Access SonarQube

Open your browser and navigate to:

```text
http://<EC2-PUBLIC-IP>:9000
```

Example:

```text
http://13.233.XXX.XXX:9000
```

---

## Default Login Credentials

```text
Username: admin
Password: admin
```

You will be prompted to change the password after the first login.

---

## Useful Docker Commands

### View Running Containers

```bash
docker ps
```

### Stop SonarQube

```bash
docker stop sonarqube
```

### Start SonarQube

```bash
docker start sonarqube
```

### Restart SonarQube

```bash
docker restart sonarqube
```

### View Logs

```bash
docker logs -f sonarqube
```

### Remove Container

```bash
docker rm -f sonarqube
```

---

## Troubleshooting

### Error: vm.max_map_count is too low

```text
max virtual memory areas vm.max_map_count [65530] is too low
```

Fix:

```bash
sudo sysctl -w vm.max_map_count=524288
sudo sysctl -p
```

### SonarQube Not Accessible

Verify:

```bash
docker ps
```

Check EC2 Security Group:

* Port 22 (SSH)
* Port 9000 (Custom TCP)

Both should be allowed from your IP or 0.0.0.0/0 as required.

---

## Verification

Check SonarQube status:

```bash
curl http://localhost:9000/api/system/status
```

Expected Output:

```json
{
  "status":"UP"
}
```

🎉 SonarQube is now successfully deployed and running on your AWS EC2 instance using Docker.

---

# ☁️ AWS EKS Setup

## Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
-o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

aws --version
```

---

## Install kubectl

```bash
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl

chmod +x kubectl

sudo mv kubectl /bin

kubectl version --output=yaml
```

---

## Install eksctl

```bash
curl --silent --location \
"https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
| tar xz -C /tmp

sudo mv /tmp/eksctl /bin

eksctl version
```

---

## Create EKS Cluster

```bash
eksctl create cluster \
--name virtualtechbox-cluster \
--region ap-south-1 \
--node-type t2.small \
--nodes 3
```

Verify:

```bash
kubectl get nodes
```

---

# 🚀 ArgoCD Installation

## Create Namespace

```bash
kubectl create namespace argocd
```

## Install ArgoCD

```bash
kubectl apply \
-n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Check Pods:

```bash
kubectl get pods -n argocd
```

---

## Install ArgoCD CLI

```bash
curl --silent --location \
-o /usr/local/bin/argocd \
https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64

chmod +x /usr/local/bin/argocd
```

---

## Expose ArgoCD UI

```bash
kubectl patch svc argocd-server \
-n argocd \
-p '{"spec":{"type":"LoadBalancer"}}'
```

Check Service:

```bash
kubectl get svc -n argocd
```

---

## Get Admin Password

```bash
kubectl get secret \
argocd-initial-admin-secret \
-n argocd -o yaml
```

Decode:

```bash
echo <BASE64_PASSWORD> | base64 --decode
```

---

# 🔗 Add EKS Cluster to ArgoCD

### Login

```bash
argocd login <ARGOCD-LOADBALANCER-DNS> \
--username admin
```

### Verify Cluster

```bash
argocd cluster list

kubectl config get-contexts
```

### Register Cluster

```bash
argocd cluster add \
<CLUSTER_CONTEXT_NAME> \
--name virtualtechbox-eks-cluster
```

---

# 🧹 Cleanup Resources

Delete Application:

```bash
kubectl delete deployment.apps/<deployment-name>

kubectl delete service/<service-name>
```

Delete EKS Cluster:

```bash
eksctl delete cluster \
--region ap-south-1 \
--name virtualtechbox-cluster
```

---

# 📹 Reference Video

https://youtu.be/e42hIYkvxoQ

---

# 🤝 Contributing

Contributions are welcome!

1. Fork the repository.
2. Create a feature branch.
3. Commit your changes.
4. Push the branch.
5. Open a Pull Request.

---

# ⭐ Support

If this project helped you learn DevOps, consider giving it a ⭐ on GitHub.

---

