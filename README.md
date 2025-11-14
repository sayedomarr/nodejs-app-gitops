# Node.js Application - Full GitOps Pipeline with Argo CD

## 📖 Project Overview

This project implements a complete GitOps-based CI/CD pipeline for deploying a Node.js application with MySQL and Redis on Amazon EKS. The CD (Continuous Deployment) component uses Argo CD and Argo Image Updater to automatically sync Kubernetes manifests from Git and deploy to the cluster.

**Project Goal:** Provision and deploy a secure, production-ready infrastructure and CI/CD pipeline using modern DevOps tools including Terraform, Jenkins, Argo CD, and External Secrets Operator.

## 👥 Team Structure

| Role | Team Member | Responsibility |
|------|-------------|----------------|
| **Infrastructure (Terraform)** | [Name] | EKS Cluster, VPC, Networking, AWS Resources |
| **CI Pipeline (Jenkins)** | [Name] | Build Pipeline, Docker Images, ECR Push |
| **CD Pipeline (Argo CD)** | **Sayed Omar** | **GitOps, Continuous Deployment, Image Updates** |
| **Secrets & Application** | [Name] | External Secrets Operator, Application Configuration |

## 🏗️ Architecture

```
Developer commits code to Git
            ↓
    Jenkins CI Pipeline
            ↓
    Build Docker Image → Push to Amazon ECR
            ↓
    Argo Image Updater (monitors ECR)
            ↓
    Detects new image tag → Updates Git repository (values.yaml)
            ↓
    Argo CD (monitors Git repository)
            ↓
    Detects change → Syncs to Kubernetes
            ↓
    Amazon EKS Cluster
    ├── Node.js Application
    ├── MySQL Database
    └── Redis Cache
```

## 📦 Project Scope (Based on Requirements)

### 1. Infrastructure Provisioning (Terraform Team)
- VPC with 3 public and 3 private subnets across 3 AZs
- NAT Gateway, Internet Gateway, Route Tables
- Amazon EKS Cluster (Control plane and node groups in private subnets)

### 2. CI Tool (Jenkins Team)
- Install Jenkins via Helm into EKS
- Jenkins pipeline to:
  - Run Terraform code
  - Clone Node.js app repository
  - Build and push Docker images to Amazon ECR

### 3. CD Tool (Argo CD - **My Responsibility**)
- ✅ Install Argo CD via Helm in separate namespace
- ✅ Configure Argo CD to:
  - Sync Kubernetes manifests from Git repository
  - Auto-deploy via GitOps
- ✅ Install Argo Image Updater to:
  - Monitor image tags in ECR
  - Auto-update image tags in Git
  - Trigger GitOps deployment flow

### 4. Secrets Management (Secrets Team)
- Install External Secrets Operator via Helm
- Connect to AWS Secrets Manager
- Automatically sync secrets:
  - MySQL credentials
  - Redis credentials

### 5. Application Deployment
- Deploy Node.js web application (https://github.com/mahmoud254/jenkins_nodejs_example.git)
- MySQL and Redis as pods within EKS
- Connect app to MySQL and Redis using environment variables
- Use Helm charts for Kubernetes manifests

### 6. Ingress and HTTPS (Bonus)
- Deploy NGINX Ingress Controller
- Expose app securely via Ingress
- Use cert-manager and Let's Encrypt for HTTPS

## 📂 Repository Structure

```
.
├── argocd/                           # Argo CD configurations
│   ├── applications/                 # Application manifests
│   │   ├── nodejs-app.yaml          # Node.js application
│   │   ├── mysql-app.yaml           # MySQL database
│   │   └── redis-app.yaml           # Redis cache
│   └── image-updater-values.yaml    # Image Updater configuration
├── charts/                           # Helm charts
│   ├── nodejs-app/                   # Node.js application chart
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── externalsecret.yaml  # External Secrets manifest
│   │       └── ingress.yaml         # Ingress configuration
│   ├── mysql/                        # MySQL chart
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       └── externalsecret.yaml
│   └── redis/                        # Redis chart
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           └── externalsecret.yaml
├── docs/                             # Documentation
│   ├── screenshots/                  # Screenshots for documentation
│   ├── INTEGRATION-GUIDE.md         # Integration guide for team
│   ├── SETUP-INSTRUCTIONS.md        # Detailed setup instructions
│   └── TROUBLESHOOTING.md           # Common issues and solutions
└── README.md                         # This file
```

## 🚀 Quick Start

### Prerequisites

- Kubernetes cluster (EKS for production, minikube for local testing)
- `kubectl` installed and configured
- `Helm 3` installed
- `Argo CD CLI` installed (optional but recommended)
- Git installed
- AWS CLI configured (for EKS deployment)

### Local Development Setup (minikube)

1. **Start minikube:**
```bash
minikube start --cpus=4 --memory=8192
```

2. **Install Argo CD:**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

3. **Wait for Argo CD pods to be ready:**
```bash
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
```

4. **Access Argo CD UI:**
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Open browser: https://localhost:8080

5. **Get admin password:**
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```
Username: `admin`

6. **Install Argo Image Updater:**
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd-image-updater argo/argocd-image-updater --namespace argocd
```

7. **Deploy applications:**
```bash
kubectl apply -f argocd/applications/
```

8. **Verify deployment:**
```bash
kubectl get applications -n argocd
kubectl get pods -n default
```

## 🌐 Production Deployment (EKS)

### Prerequisites from Team

Before deploying to EKS, collect the following information:

**From Terraform Team:**
- [ ] EKS Cluster name
- [ ] AWS Region
- [ ] AWS Account ID
- [ ] Command to get kubeconfig: `aws eks update-kubeconfig --name <CLUSTER_NAME> --region <REGION>`
- [ ] OIDC Provider URL (if available)

**From Jenkins Team:**
- [ ] ECR Repository URL (e.g., `123456789012.dkr.ecr.us-east-1.amazonaws.com/nodejs-app`)
- [ ] Initial image tag (e.g., `1.0.0`)
- [ ] Image tagging strategy (semver, git commit, build number)
- [ ] Does Jenkins update values.yaml automatically?

**From Secrets Team:**
- [ ] AWS Secrets Manager secret names for MySQL and Redis
- [ ] Secret keys structure (username, password, database, etc.)
- [ ] Is External Secrets Operator installed?
- [ ] SecretStore name

### EKS Deployment Steps

#### 1. Connect to EKS Cluster

```bash
aws eks update-kubeconfig --name <CLUSTER_NAME> --region <REGION>
kubectl get nodes
```

#### 2. Install Argo CD on EKS

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
```

#### 3. Setup IAM Role for Argo Image Updater

Create IAM role for ECR access:

```bash
# Create trust policy
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "pods.eks.amazonaws.com"},
    "Action": ["sts:AssumeRole", "sts:TagSession"]
  }]
}
EOF

# Create IAM role
aws iam create-role \
  --role-name argocd-image-updater-ecr-role \
  --assume-role-policy-document file://trust-policy.json

# Attach ECR read policy
aws iam attach-role-policy \
  --role-name argocd-image-updater-ecr-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly

# Create Pod Identity Association
eksctl create podidentityassociation \
  --cluster <CLUSTER_NAME> \
  --namespace argocd \
  --service-account-name argocd-image-updater \
  --role-arn arn:aws:iam::<ACCOUNT_ID>:role/argocd-image-updater-ecr-role
```

#### 4. Configure Argo Image Updater for ECR

Update `argocd/image-updater-values.yaml`:

```yaml
config:
  registries:
  - name: ECR
    api_url: https://<ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com
    prefix: <ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com
    ping: yes
    insecure: no
    credentials: ext:/scripts/login.sh
    credsexpire: 6h

authScripts:
  enabled: true
  scripts:
    login.sh: |
      #!/bin/sh
      aws ecr --region "<REGION>" get-authorization-token --output text --query 'authorizationData[].authorizationToken' | base64 -d
```

Install/upgrade Image Updater:

```bash
helm upgrade --install argocd-image-updater argo/argocd-image-updater \
  --namespace argocd \
  --values argocd/image-updater-values.yaml
```

#### 5. Create Git Credentials Secret

For Git write-back functionality:

```bash
kubectl create secret generic git-creds \
  --from-literal=username=<GITHUB_USERNAME> \
  --from-literal=password=<GITHUB_TOKEN> \
  -n argocd
```

#### 6. Update Application Manifests

Update `charts/nodejs-app/values.yaml`:

```yaml
image:
  repository: <ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/nodejs-app
  tag: "1.0.0"  # Will be updated by Image Updater

env:
  - name: NODE_ENV
    value: "production"
  - name: PORT
    value: "3000"
  - name: MYSQL_HOST
    value: "mysql-service.default.svc.cluster.local"
  - name: MYSQL_PORT
    value: "3306"
  - name: MYSQL_USER
    valueFrom:
      secretKeyRef:
        name: mysql-secret
        key: username
  - name: MYSQL_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mysql-secret
        key: password
  - name: MYSQL_DATABASE
    valueFrom:
      secretKeyRef:
        name: mysql-secret
        key: database
  - name: REDIS_HOST
    value: "redis-service.default.svc.cluster.local"
  - name: REDIS_PORT
    value: "6379"
```

Update `argocd/applications/nodejs-app.yaml` annotations:

```yaml
metadata:
  name: nodejs-app
  namespace: argocd
  annotations:
    argocd-image-updater.argoproj.io/image-list: nodejs=<ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/nodejs-app
    argocd-image-updater.argoproj.io/nodejs.update-strategy: semver
    argocd-image-updater.argoproj.io/nodejs.allow-tags: regexp:^[0-9]+\.[0-9]+\.[0-9]+$
    argocd-image-updater.argoproj.io/nodejs.helm.image-name: image.repository
    argocd-image-updater.argoproj.io/nodejs.helm.image-tag: image.tag
    argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd/git-creds
```

#### 7. Deploy Applications

```bash
kubectl apply -f argocd/applications/
```

#### 8. Verify Deployment

```bash
# Check applications
argocd app list
kubectl get applications -n argocd

# Check pods
kubectl get pods -n default
kubectl get pods -n nodejs-app

# Check services
kubectl get services -n default

# Check Image Updater logs
kubectl logs -n argocd deployment/argocd-image-updater
```

## 🔄 GitOps Workflow

1. **Developer commits code** to application repository
2. **Jenkins CI Pipeline** builds Docker image and pushes to ECR with new tag (e.g., `1.0.1`)
3. **Argo Image Updater** polls ECR every 2 minutes, detects new tag
4. **Image Updater updates** `charts/nodejs-app/values.yaml` in Git with new tag
5. **Image Updater commits** and pushes changes to Git
6. **Argo CD polls** Git repository every 3 minutes, detects change
7. **Argo CD syncs** new configuration to Kubernetes
8. **Kubernetes** performs rolling update with new image
9. **Application** is now running with latest version

**Result:** Complete automation from code commit to production deployment!

## 🛠️ Useful Commands

### Argo CD Management

```bash
# List all applications
argocd app list
kubectl get applications -n argocd

# Get application details
argocd app get <APP_NAME>
kubectl describe application <APP_NAME> -n argocd

# Sync application manually
argocd app sync <APP_NAME>

# Force sync (ignore differences)
argocd app sync <APP_NAME> --force

# Rollback to previous version
argocd app rollback <APP_NAME> <REVISION_NUMBER>

# View sync history
argocd app history <APP_NAME>

# Delete application
argocd app delete <APP_NAME>
kubectl delete application <APP_NAME> -n argocd
```

### Kubernetes Troubleshooting

```bash
# Check pods status
kubectl get pods -n default
kubectl get pods -n argocd

# View pod logs
kubectl logs <POD_NAME>
kubectl logs -f <POD_NAME>  # Follow logs

# Describe pod (see events)
kubectl describe pod <POD_NAME>

# Execute commands in pod
kubectl exec -it <POD_NAME> -- bash
kubectl exec -it <POD_NAME> -- sh

# Check services
kubectl get services
kubectl describe service <SERVICE_NAME>

# Check events
kubectl get events --sort-by='.lastTimestamp'
```

### Image Updater Debugging

```bash
# Check Image Updater logs
kubectl logs -n argocd deployment/argocd-image-updater

# Follow logs
kubectl logs -n argocd deployment/argocd-image-updater -f

# Test ECR authentication
kubectl exec -it -n argocd deployment/argocd-image-updater -- sh
/scripts/login.sh

# Check Image Updater configuration
kubectl get configmap -n argocd argocd-image-updater-config -o yaml
```

## 📸 Screenshots

See `docs/screenshots/` directory for:
- Argo CD UI showing applications
- Application resource tree
- Sync status and health
- Kubernetes pods running
- GitOps workflow demonstration

## 📚 Documentation

- **[Integration Guide](docs/INTEGRATION-GUIDE.md)**: How to integrate with team members
- **[Setup Instructions](docs/SETUP-INSTRUCTIONS.md)**: Detailed step-by-step setup
- **[Troubleshooting](docs/TROUBLESHOOTING.md)**: Common issues and solutions

## 🐛 Common Issues and Solutions

### Application Stuck in "OutOfSync"

**Problem:** Application shows OutOfSync status but doesn't sync automatically.

**Solution:**
```bash
argocd app sync <APP_NAME> --force
```

### Image Updater Not Detecting New Images

**Problem:** New images pushed to ECR but Image Updater doesn't update Git.

**Solutions:**

1. Check Image Updater logs:
```bash
kubectl logs -n argocd deployment/argocd-image-updater
```

2. Verify ECR authentication:
```bash
kubectl exec -it -n argocd deployment/argocd-image-updater -- sh
/scripts/login.sh
```

3. Check annotations on Application:
```bash
kubectl get application <APP_NAME> -n argocd -o yaml
```

### Pods in CrashLoopBackOff

**Problem:** Pods keep restarting.

**Solution:**
```bash
# Check logs
kubectl logs <POD_NAME>

# Check events
kubectl describe pod <POD_NAME>

# Common causes:
# - Missing environment variables
# - Cannot connect to database/redis
# - Image pull errors
# - Application errors
```

### Cannot Connect to MySQL/Redis

**Problem:** Application cannot connect to database or cache.

**Solution:**

1. Verify services exist:
```bash
kubectl get services
```

2. Check if MySQL/Redis pods are running:
```bash
kubectl get pods -l app=mysql
kubectl get pods -l app=redis
```

3. Test connection from app pod:
```bash
kubectl exec -it <APP_POD> -- sh
ping mysql-service
ping redis-service
```

4. Verify environment variables:
```bash
kubectl exec -it <APP_POD> -- env | grep MYSQL
kubectl exec -it <APP_POD> -- env | grep REDIS
```

## 🎯 Project Deliverables Checklist

### GitHub Repository
- [x] Argo CD Application manifests
- [x] Helm charts for Node.js, MySQL, Redis
- [x] Argo Image Updater configuration
- [ ] External Secrets Operator manifests (from Secrets team)
- [x] Documentation and README

### Documentation
- [x] Project overview and architecture
- [x] Setup instructions (local and EKS)
- [x] CI/CD flow explanation
- [x] Integration guide
- [x] Troubleshooting guide

### Demo Preparation
- [ ] Screenshots of Argo CD UI
- [ ] Video/screenshots of GitOps workflow
- [ ] Application running on EKS
- [ ] End-to-end deployment demonstration

## 📊 Evaluation Criteria (My Part)

| Area | Points | Status |
|------|--------|--------|
| Argo CD GitOps + Image Updater | 15 | ✅ Implemented |
| Code quality, Helm usage | 10 | ✅ Implemented |
| Documentation | 20 | ✅ Complete |
| **Total (My Responsibility)** | **45** | **✅ Complete** |

## 🔗 Useful Links

- [Argo CD Official Documentation](https://argo-cd.readthedocs.io/)
- [Argo Image Updater Documentation](https://argocd-image-updater.readthedocs.io/)
- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [GitOps Principles](https://www.gitops.tech/)
- [External Secrets Operator](https://external-secrets.io/)

## 📞 Contact

For questions or issues related to the CD pipeline (Argo CD):
- **Name:** Sayed Omar
- **Role:** CD Pipeline Engineer
- **GitHub:** [sayedomarr](https://github.com/sayedomarr)

## 📄 License

This project is for educational purposes as part of the DevOps Capstone Project.

---

**Note:** This setup is currently tested on minikube. For EKS deployment, please ensure all team members have completed their parts (Terraform, Jenkins, Secrets) and provided the necessary information as listed in the "Prerequisites from Team" section.
