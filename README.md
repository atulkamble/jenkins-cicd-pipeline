Here’s a **complete Jenkins CI/CD pipeline** project integrating:

* **Git** for source code versioning
* **Docker** for image creation
* **Kubernetes** for deployment
* **Terraform** for infrastructure provisioning

---

## ✅ Tech Stack Used

| Tool       | Purpose                           |
| ---------- | --------------------------------- |
| GitHub     | Source code repository            |
| Jenkins    | CI/CD Pipeline                    |
| Docker     | Build & push image                |
| Terraform  | Provision cloud infra (e.g., AWS) |
| Kubernetes | Application deployment            |
| Helm (opt) | Template Kubernetes manifests     |

---

## 🗂 Project Structure

```
.
├── Jenkinsfile
├── Dockerfile
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── app/
│   └── main.py
├── requirements.txt
```

---

## 🧾 Jenkinsfile – Full Pipeline

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yourdockerhubuser/appname:${BUILD_NUMBER}"
        KUBE_CONFIG = credentials('kubeconfig-cred-id') // Jenkins secret text or file
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/youruser/yourrepo.git'
            }
        }

        stage('Terraform Init & Apply') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push $DOCKER_IMAGE"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred-id', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed"
        }
    }
}
```

---

## 🐳 Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app/ .
CMD ["python", "main.py"]
```

---

## 📦 Sample `main.py`

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from Jenkins Pipeline!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

---

## 📜 Terraform (AWS EC2 or EKS example)

Basic EC2 example:

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c94855ba95c71c99"
  instance_type = "t2.micro"
  tags = {
    Name = "JenkinsDeployedEC2"
  }
}
```

---

## ☸️ Kubernetes Deployment Example

### `k8s/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: jenkins-app
  template:
    metadata:
      labels:
        app: jenkins-app
    spec:
      containers:
      - name: jenkins-app
        image: yourdockerhubuser/appname:latest
        ports:
        - containerPort: 5000
```

### `k8s/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
spec:
  selector:
    app: jenkins-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
```

---

## 🔐 Jenkins Credentials Required

1. `dockerhub-creds`: DockerHub username and password (username/password)
2. `kubeconfig-cred-id`: Kubeconfig file (file credential)
3. GitHub: Can be public or via SSH key
4. AWS Credentials: If Terraform deploys on AWS, configure IAM or `aws configure`

---

## 🛠 Jenkins Plugins to Install

* Git
* Docker Pipeline
* Kubernetes CLI Plugin
* Terraform Plugin (optional)
* Credentials Binding Plugin
* Pipeline Utility Steps

---
