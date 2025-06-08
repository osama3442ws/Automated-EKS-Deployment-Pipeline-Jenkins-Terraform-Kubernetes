
---

# 🚀 End-to-End CI/CD Pipeline on AWS using Jenkins, Terraform, and EKS

This project sets up a complete CI/CD pipeline using **Jenkins**, **Terraform**, and **Amazon EKS (Kubernetes)** on **AWS Cloud**. The system allows developers to automatically build, test, analyze, and deploy containerized applications in a secure and scalable way.

---

## 🧱 Architecture Overview

The system is divided into the following components:

### 🔨 1. Infrastructure Provisioning using Terraform

**Goal:**
Provision all required AWS resources using Terraform.

**Steps:**

* Developers write Terraform scripts locally.
* Run `terraform apply` to deploy the infrastructure on AWS.
* Resources created include:

  * Two Virtual Private Clouds (VPCs)
  * Jenkins Server on an EC2 instance
  * Amazon EKS Cluster (Kubernetes)
  * Private subnets for EKS worker nodes
  * Application Load Balancer (ALB)

---

### 🧰 2. Jenkins Server Setup

**Goal:**
Set up Jenkins on an EC2 instance with all necessary tools.

**Installed tools:**

* Jenkins
* Docker
* Terraform CLI
* `kubectl`
* Helm
* SonarQube
* Trivy (for security scanning)

---

### 🔗 3. GitHub Integration

**Goal:**
Connect Jenkins to GitHub to automate code fetching and builds.

**Steps:**

* Store your application code and Kubernetes manifests in a GitHub repository.
* Create a `Jenkinsfile` defining pipeline stages.
* Configure GitHub Webhooks and Jenkins credentials for automation.

---

### 🚀 4. Jenkins CI/CD Pipeline

**Goal:**
Automate the entire build and deployment process.

**Pipeline Stages:**

1. Checkout source code from GitHub
2. Build Docker image
3. Scan the image with Trivy
4. Run tests (e.g., Unit tests)
5. Analyze code with SonarQube
6. Push Docker image to Amazon ECR (optional)
7. Deploy to EKS using `kubectl apply`

**Sample `Jenkinsfile`:**

```groovy
pipeline{
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "us-east-1"
    }
    stages {
        stage('Checkout SCM') {
            steps {
                script {
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/vishal2505/terraform-eks-cicd.git']])
                }
            }
        }
        stage('Initializing Terraform'){
            steps {
                script {
                    dir('tf-aws-eks'){
                        sh 'terraform init'
                    }
                }
            }
        }
        stage('Validating Terraform'){
            steps {
                script {
                    dir('tf-aws-eks'){
                        sh 'terraform validate'
                    }
                }
            }
        }
        stage('Terraform Plan'){
            steps {
                script {
                    dir('tf-aws-eks'){
                        sh 'terraform plan -var-file=variables/dev.tfvars'
                    }
                    input(message: "Are you sure to proceed?", ok: "Proceed")
                }
            }
        }
        stage('Creating/Destroying EKS Cluster'){
            steps {
                script {
                    dir('tf-aws-eks'){
                        sh 'terraform $action -var-file=variables/dev.tfvars -auto-approve' 
                    }
                }
            }
        }
        stage('Deploying Nginx Application') {
            steps{
                script{
                    dir('manifest') {
                        sh 'aws eks update-kubeconfig --name my-eks-cluster'
                        sh 'kubectl create namespace eks-nginx-app'
                        sh 'kubectl apply -f deployment.yaml -n eks-nginx-app'
                        sh 'kubectl apply -f service.yaml -n eks-nginx-app'
                    }
                }
            }
        }
    }
}
```

---

### ☸️ 5. EKS & Kubernetes Deployment

**Goal:**
Run the application inside a Kubernetes cluster managed by Amazon EKS.

**Steps:**

* Jenkins deploys manifests (`Deployment`, `Service`, `Ingress`, etc.) to the EKS cluster.
* EKS runs pods inside private subnets across worker nodes.

---

### 🌐 6. Access via Application Load Balancer (ALB)

**Goal:**
Allow users to access the application from the internet.

**How it works:**

* Users send requests via the ALB's public DNS.
* The ALB routes traffic to the correct Kubernetes `Service`.
* The service routes it to the running `Pod`.

---

## 🧠 Summary

| Component      | Description                        |
| -------------- | ---------------------------------- |
| Developer      | Writes Terraform and pipeline code |
| Jenkins Server | Manages CI/CD pipeline and tools   |
| GitHub         | Stores source code and Jenkinsfile |
| Amazon EKS     | Runs the application in Kubernetes |
| ALB            | Handles user traffic securely      |

---

