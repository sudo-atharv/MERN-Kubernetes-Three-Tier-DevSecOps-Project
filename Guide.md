# 🚀 MERN Stack Three-Tier DevSecOps Project on AWS EKS

## 📌 Overview

This repository demonstrates a **production-grade, end-to-end MERN Stack deployment** on **AWS EKS**, implemented using **enterprise DevOps and DevSecOps practices**. The project follows a **three-tier architecture** (Frontend, Backend, Database) and uses **CI/CD automation, GitOps, Infrastructure as Code, security scanning, observability, and DNS-based traffic routing** — closely aligned with real corporate environments.

---

## 🏗️ High-Level Architecture

* **Frontend**: React application
* **Backend**: Node.js / Express API
* **Database**: MongoDB
* **Container Orchestration**: Amazon EKS
* **CI/CD**: Jenkins + GitHub
* **GitOps CD**: Argo CD
* **IaC**: Terraform
* **Security**: SonarQube, Trivy
* **Container Registry**: Amazon ECR
* **Ingress & DNS**: AWS Load Balancer Controller + Route 53
* **Monitoring**: Prometheus & Grafana

---

## 🔧 Tooling & Stack

| Category   | Tools                              |
| ---------- | ---------------------------------- |
| Cloud      | AWS (EKS, EC2, ECR, Route 53, IAM) |
| CI         | Jenkins                            |
| CD         | Argo CD (GitOps)                   |
| IaC        | Terraform                          |
| Containers | Docker, Kubernetes                 |
| Security   | SonarQube, Trivy                   |
| Networking | ALB Ingress Controller             |
| Monitoring | Prometheus, Grafana                |

---

## 🖥️ Jenkins Server Setup

* EC2 Instance: `t3a.large`
* Open ports: `22, 80, 8080, 9000`

### 🔹 Install Java

```bash
sudo apt update -y
sudo apt install openjdk-17-jre -y
sudo apt install openjdk-17-jdk -y
java --version
```

### 🔹 Install Jenkins

```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update -y
sudo apt install jenkins -y
```

Access Jenkins:

```
http://<jenkins-public-ip>:8080
```

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 🔹 Install Docker

```bash
sudo apt update -y
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
sudo chmod 777 /var/run/docker.sock
```

### 🔹 Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
```

### 🔹 Install kubectl

```bash
curl -LO https://dl.k8s.io/release/v1.28.4/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

### 🔹 Install eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### 🔹 Install Terraform

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update -y
sudo apt install terraform -y
```

### 🔹 Install Trivy

```bash
sudo apt install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt update -y
sudo apt install trivy -y
```

### 🔹 Install Helm

```bash
sudo snap install helm --classic
```

### 🔹 Run SonarQube

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

---

## ☁️ Infrastructure Provisioning (IaC)

* AWS infrastructure provisioned using **Terraform**
* Terraform executed via **Jenkins pipeline**
* EKS cluster creation automated

📄 Jenkinsfile (Infra):
[https://github.com/sudo-atharv/EKS-Terraform-GitHub-Actions/blob/master/Jenkinsfile](https://github.com/sudo-atharv/EKS-Terraform-GitHub-Actions/blob/master/Jenkinsfile)

---

## 🔐 CI Pipeline – DevSecOps Flow

Separate Jenkins pipelines are created for **frontend** and **backend**.

### CI Stages

1. Source code checkout (GitHub)
2. SonarQube code quality analysis
3. Quality Gate validation
4. Trivy filesystem scan
5. Docker image build
6. Push image to Amazon ECR
7. Trivy image vulnerability scan
8. Auto-update Kubernetes manifests (image tag)

🔹 Image updates are committed back to GitHub → triggering Argo CD sync.

---

## 🧪 Security Integration

* **SonarQube**: Static code analysis & quality gates
* **Trivy FS Scan**: Dependency & config vulnerabilities
* **Trivy Image Scan**: Container image CVEs

---

## 🔁 GitOps with Argo CD

### Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Expose Argo CD Server

```bash
kubectl get svc -n argocd
kubectl edit svc argocd-server -n argocd
```

Change service type from `ClusterIP` to `LoadBalancer`.

Get external URL:

```bash
kubectl get svc -n argocd
```

### Get Argo CD Admin Password

```bash
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode
```

Login:

* Username: `admin`
* Password: decoded value

### Add Repository to Argo CD

* Settings → Repositories → Connect Repo
* Protocol: HTTPS
* Provide GitHub PAT if repository is private

### Create Applications

Applications are created for Database, Backend, Frontend, and Ingress with:

* Sync Policy: **Automatic**
* Self Heal: **Enabled**

---

## 🌐 Ingress & DNS Routing

### Install AWS Load Balancer Controller

#### Add Helm Repo

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

#### Install Controller

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=<cluster-name> \
--set serviceAccount.create=false \
--set serviceAccount.name=aws-load-balancer-controller
```

Verify:

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

### Ingress & Route 53

* Kubernetes Ingress creates an **AWS ALB**
* Create Route 53 Hosted Zone
* Update NS records in domain registrar
* Create **A-record → Alias → ALB**

Traffic Flow:

```
Route53 → ALB (Ingress) → Service → Pods
```

---

## 📊 Monitoring & Observability

### Install Prometheus (Helm)

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus
```

Expose Prometheus:

```bash
kubectl edit svc prometheus-server
```

Change type to `LoadBalancer`.

---

### Install Grafana (Helm)

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana
```

Expose Grafana:

```bash
kubectl edit svc grafana
```

Change type to `LoadBalancer`.

Get admin password:

```bash
kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

Login:

* Username: `admin`
* Password: decoded value

---

---

## 🛠️ Troubleshooting Guide

### AWS Load Balancer Controller Not Running

```bash
kubectl get pods -n kube-system | grep aws-load-balancer
```

If failing, ensure VPC ID is passed:

```bash
kubectl edit deployment aws-load-balancer-controller -n kube-system
```

Add:

```
--aws-vpc-id=<vpc-id>
```

### ImagePullBackOff

* Verify image tag updated
* Check ECR permissions
* Validate image exists in ECR

### Argo CD App OutOfSync

* Ensure correct repo path
* Check namespace exists

```bash
kubectl create ns three-tier
```

### Domain Not Resolving

* Verify Route53 A-record
* Check NS propagation
* Wait 10–15 minutes

---

## 📂 Repository Structure

```
├── Application-Code/
│   ├── frontend/
│   └── backend/
├── Kubernetes-Manifests-file/
│   ├── Frontend/
│   ├── Backend/
│   ├── Database/
│   └── ingress.yml
├── Terraform/
└── README.md
```

---


