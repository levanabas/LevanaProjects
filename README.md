# Pacman DevOps Project 🎮

A full DevOps pipeline for building and deploying the **Pacman** application using:
- **AWS EKS (Kubernetes)**
- **ECR (Elastic Container Registry)**
- **GitHub Actions (CI/CD)**
- **Docker**

---

## 🚀 Progress so far

### 1. EKS Cluster creation
Cluster configuration file (`EKS_Cluster/cluster.yaml`):
```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: levana-cluster
  region: us-west-2
  version: "1.29"

autoModeConfig:
  enabled: true

1. Command:
eksctl create cluster -f EKS_Cluster/cluster.yaml

2. Connect kubectl:
aws eks update-kubeconfig --region us-west-2 --name levana-cluster
kubectl get nodes

3. Dockerfile:
pacman/docker/Dockerfile

4. Build and push Docker image to ECR

Repository created in ECR: pacman

Authenticate:
aws ecr get-login-password --region us-west-2 \
  | docker login --username AWS --password-stdin 601440149085.dkr.ecr.us-west-2.amazonaws.com

Build and push:
docker build -t pacman -f pacman/docker/Dockerfile pacman/
docker tag pacman:latest 601440149085.dkr.ecr.us-west-2.amazonaws.com/pacman:latest
docker push 601440149085.dkr.ecr.us-west-2.amazonaws.com/pacman:latest

5. GitHub Actions Workflow (cicd.yml)

name: Build and Push to ECR

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-west-2

      - name: Build, tag, and push Docker image
        run: |
          aws ecr get-login-password --region us-west-2 \
            | docker login --username AWS --password-stdin 601440149085.dkr.ecr.us-west-2.amazonaws.com

          docker build -t pacman -f pacman/docker/Dockerfile pacman/
          docker tag pacman:latest 601440149085.dkr.ecr.us-west-2.amazonaws.com/pacman:latest
          docker push 601440149085.dkr.ecr.us-west-2.amazonaws.com/pacman:latest

