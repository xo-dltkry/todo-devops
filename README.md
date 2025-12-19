# DevOps Final Project – Spring Boot ToDo Application

This project is a continuation of **minitask1** and **minitask2**, combining them into a full DevOps workflow:

> **Linux VM → Git → Docker → Jenkins (CI/CD) → Kubernetes → Ansible**

The goal is to demonstrate not only how tools are used, but also **why** specific DevOps decisions are made, with a focus on **security**, **scalability**, **automation**, and **reproducibility**.

---

## 1. Architecture Diagram

### 1.1 High-level DevOps Architecture

```mermaid
flowchart LR
    Dev[Developer\n(local)] -->|git push| GitRepo[(GitHub/GitLab)]

    subgraph VM[Linux VM (Ubuntu 22.04)]
        subgraph Provision[Provisioned via Ansible]
            DockerEngine[(Docker)]
            JenkinsServer[Jenkins]
            Minikube[(Minikube / Kubernetes)]
        end
    end

    GitRepo -->|webhook / polling| JenkinsServer

    JenkinsServer -->|build & test\n(Maven)| BuildArtifacts[(JAR)]
    JenkinsServer -->|build & push image| DockerRegistry[(Docker Registry)]

    JenkinsServer -->|kubectl apply\n(k8s manifests)| K8sCluster[(Kubernetes Cluster in Minikube)]

    subgraph K8s[Kubernetes Namespace: todo]
        AppDep[Deployment: todo-app]
        AppSvc[Service: todo-service]
        ConfigMapCM[ConfigMap: configmap_Dauletkerey]
        SecretSec[Secret: secret_Dauletkerey]
        DB[(PostgreSQL DB)]
    end

    AppDep --> DB
    ConfigMapCM --> AppDep
    SecretSec --> AppDep

    UserBrowser[User Browser / REST Client] -->|HTTP / JSON| AppSvc --> AppDep
Explanation:

A Linux VM hosts Docker, Jenkins, Minikube/Kubernetes and is provisioned via Ansible.

Source code is stored in GitHub/GitLab.

Jenkins pulls the code, builds/tests it, builds a Docker image, pushes it to a registry and deploys to Kubernetes.

The application runs as a Spring Boot REST API with a PostgreSQL database. Configuration comes from ConfigMap and Secret.

2. Application Architecture
Backend: Spring Boot (ToDo REST API)

Full CRUD operations for tasks

Input validation and global exception handling

Spring Boot Actuator health endpoints (/actuator/health)

Database: PostgreSQL

Containerization: Docker + Docker Compose

Orchestration: Kubernetes (Deployment, Service, ConfigMap, Secret, HPA)

Automation:

Jenkins – CI/CD pipeline

Ansible – provisioning Linux VM, Docker, Jenkins, Minikube, and deploying k8s manifests

3. Prerequisites
Linux VM (Ubuntu 20.04+ / 22.04+)

Non-root user with sudo (e.g. devops)

SSH access via SSH keys (password login disabled)

Firewall (ufw) configured: allow SSH, HTTP, HTTPS

Installed tools on the VM (either manually or via Ansible):

git

openjdk-17-jdk

docker + docker-compose

kubectl

minikube or kind

ansible

jenkins

4. Project Structure

/opt/devops-project/
├── app/                  # Spring Boot application (ToDo)
├── docker/               # Dockerfile and docker-compose
├── k8s/                  # Kubernetes manifests
├── ansible/              # Ansible inventory, roles, and playbooks
└── scripts/              # Helper scripts (install, verify, etc.)
File naming convention (example):

docker/Dockerfile_Dauletkerey

docker/docker-compose_Dauletkerey.yml

k8s/deployment_Dauletkerey.yaml

k8s/service_Dauletkerey.yaml

k8s/configmap_Dauletkerey.yaml

k8s/secret_Dauletkerey.yaml

Jenkinsfile_Dauletkerey

ansible/site_Dauletkerey.yml

ansible/inventory_Dauletkerey.ini

5. Step-by-Step Setup Instructions
5.1 Linux VM & Git (Part 0)
Create VM in VirtualBox / cloud:

OS: Ubuntu Server / Desktop 22.04+

User: devops with sudo

Configure SSH & firewall:

# Add your SSH public key to ~/.ssh/authorized_keys
```
sudo ufw allow OpenSSH
sudo ufw enable
```
Create directory structure:

```
sudo mkdir -p /opt/devops-project/{app,docker,k8s,ansible,scripts}
sudo chown -R devops:devops /opt/devops-project
```
Clone your Git repository:

```
cd /opt/devops-project
git clone https://github.com/<your-username>/DevOps_mini_task2.git app
```
Branching model:

main – production-ready code

develop – integration

feature/* – feature branches

Example:

```
git checkout -b feature/add-todo-endpoints
# work...
git commit -m "feat(todo): add CRUD endpoints"
git checkout develop
git merge feature/add-todo-endpoints
git tag v1.0.0
git push --all
git push --tags
```
5.2 Build & Run with Docker (Part 1)
Below assumes Maven build. If you use Gradle, replace commands accordingly.

Build JAR:

```
cd /opt/devops-project/app
mvn clean package -DskipTests=false
```
# Result: target/todo-app-*.jar
Build Docker image (multi-stage) using Dockerfile_Dauletkerey:

```
cd /opt/devops-project/docker
docker build -f Dockerfile_Dauletkerey \
  -t tg/todo-app:1.1 \
  ..
```
Analyze image:

```
docker history tg/todo-app:1.1
docker images | grep todo-app
```
Run via Docker Compose:

Create .env_Dauletkerey:

```
POSTGRES_DB=todo
POSTGRES_USER=todo
POSTGRES_PASSWORD=todo
APP_TAG=1.1
```
Start stack:

```
docker-compose -f docker-compose_Dauletkerey.yml up -d
docker ps
```
5.3 Jenkins CI/CD Setup (Part 2)
Install Jenkins on the VM (not in Docker).

Open Jenkins UI (http://<VM_IP>:8080), unlock, create admin user, disable anonymous access.

Install plugins:

Pipeline

Git

Docker

Credentials Binding

Configure credentials:

GitHub token (ID: github-token)

Docker registry credentials (ID: dockerhub-creds, or similar)

Create Pipeline job (e.g. todo-app-pipeline):

Pipeline from SCM

SCM: Git

Repo URL: https://github.com/<your-username>/DevOps_mini_task2.git

Script Path: Jenkinsfile_Dauletkerey

Trigger build:

On git push to main or manually.

The pipeline should:

Check out code

Build & test (Maven)

Build Docker image with dynamic tag (commit hash / build number)

Push image to registry

Deploy to Kubernetes (via kubectl apply ...)

Archive artifacts & print notifications to console

5.4 Kubernetes Deployment (Part 3)
Start Minikube (example):
```
minikube start --driver=docker
```
Create namespace and apply manifests:

```
kubectl create namespace todo

kubectl apply -n todo -f k8s/configmap_Dauletkerey.yaml
kubectl apply -n todo -f k8s/secret_Dauletkerey.yaml
kubectl apply -n todo -f k8s/deployment_Dauletkerey.yaml
kubectl apply -n todo -f k8s/service_Dauletkerey.yaml
```
Enable HPA (Horizontal Pod Autoscaler):
```
kubectl apply -n todo -f k8s/hpa_Dauletkerey.yaml
```
Check pods and services:

```
kubectl get pods -n todo
kubectl get svc -n todo
kubectl get hpa -n todo
```
5.5 Ansible Automation (Part 4)
Inventory (example: ansible/inventory_Dauletkerey.ini):

```
webserver2 ansible_host=<VM_IP> ansible_user=devops
```

[devops_vm:vars]
ansible_python_interpreter=/usr/bin/python3
Run site playbook (installs Docker, Jenkins, Minikube, deploys k8s manifests):

```
cd /opt/devops-project/ansible
ansible-playbook -i inventory_Dauletkerey.ini site_Dauletkerey.yml
Sensitive variables (e.g. DB password) are stored in group_vars and encrypted via Ansible Vault:
```

```
ansible-vault create group_vars/all/vault_Dauletkerey.yml
```
6. CI/CD Pipeline Flow Description
6.1 Pipeline Stages (Conceptual)
mermaid

flowchart LR
    Dev[Developer] -->|commit & push| GitRepo[(GitHub)]
    GitRepo -->|webhook / polling| Jenkins[Jenkins Pipeline]

    subgraph Jenkins Pipeline
        A[Checkout SCM] --> B[Build & Test\n(mvn clean test)]
        B --> C[Build JAR\n(mvn package)]
        C --> D[Build Docker Image\n(tg/todo-app:<tag>)]
        D --> E[Push Image\nto Docker Registry]
        E --> F[Deploy to Kubernetes\n(kubectl apply ...)]
        F --> G[Post Actions\n(archive artifacts, notify)]
    end

    Jenkins --> K8s[(Kubernetes Cluster)]
    User[End User / REST Client] -->|HTTP| K8s
6.2 Stage Details
Checkout

Uses Jenkins Git plugin and stored Git credentials.

Build & Test

Runs mvn clean test.

If tests fail → pipeline fails immediately.

Package Application

Runs mvn package.

Outputs JAR in target/*.jar.

Docker Build & Tag

docker build -f docker/Dockerfile_Dauletkerey -t tg/todo-app:${BUILD_TAG} .

BUILD_TAG may be based on BUILD_NUMBER or GIT_COMMIT.

Push Docker Image

Logs into Docker registry using Jenkins credentials.

Pushes tg/todo-app:${BUILD_TAG}.

If push fails → pipeline fails.

Deploy to Kubernetes

Uses shell step in Jenkins:

```
kubectl apply -n todo -f k8s/configmap_Dauletkerey.yaml
kubectl apply -n todo -f k8s/secret_Dauletkerey.yaml
kubectl apply -n todo -f k8s/deployment_Dauletkerey.yaml
kubectl apply -n todo -f k8s/service_Dauletkerey.yaml
kubectl apply -n todo -f k8s/hpa_Dauletkerey.yaml
```
Optionally performs rolling update:

```
kubectl rollout status deployment todo-app -n todo
```
Post-Build Actions

Archive JAR or logs.
