
---

# Kubernetes Hackathon

This project demonstrates a containerized microservice architecture deployed on a local Kubernetes cluster (Minikube). The entire system runs inside a VirtualBox VM using Ubuntu. This guide will walk you through setting up the environment, deploying the application, and exploring different solutions.

## Requirements

### Host Machine:

* **VirtualBox**: Ensure it's installed on your host machine.
*  **UBUNTU-24.04.2**: This is the operating system of the virtual machine.
* **Internet Connection**: Required for installing dependencies inside the Ubuntu VM.

### Inside Ubuntu VM (20.04 or later):

* Docker
* Minikube
* Kubectl
* Git

To clone our repo
    - Open a terminal (PowerShell on Windows; any shell on Linux/macOS) 
    - Run: `git clone https://github.com/kubernetes_hackathon.git`
  


## Setup Instructions

### 1. Update System & Install Dependencies

Run the following commands to update your system:

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Docker

Install Docker and add your user to the Docker group:

```bash
sudo apt install -y docker.io
sudo usermod -aG docker $USER
newgrp docker
```

### 3. Install Minikube

Download and install Minikube:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### 4. Install Kubectl

Download and install kubectl:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### 5. Start Minikube

Start Minikube with the Docker driver:

```bash
minikube start --driver=docker
```

> **Note**: If using VirtualBox driver, replace `--driver=docker` with `--driver=virtualbox`.

### 6. Clone the Project Repository

Clone the repository and navigate into the project directory:

```bash
git clone https://github.com/yourusername/your-repo.git
cd your-repo
```

### 7. Deploy to Kubernetes

Deploy the application using the Kubernetes configuration files:

```bash
kubectl apply -f k8s/
```

### 8. Access the Application

To access the application, you can retrieve the service URL as follows:

```bash
minikube service <service-name> --url
```

## Solutions Overview

### Solution 1: Basic Deployment

Widgetario, a company that sells gadgets, runs its public web app on Kubernetes. All components are packaged into container images and published on Docker Hub. In this solution, we start with a simple setup using one replica for each component. We were able to ping `127.0.0.1` or browse port `8080` on the cluster.

### Solution 2: Secure Configuration

The `widgetario/products-db:21.03` image exposes the database password through environment variables. This solution addresses the security issue by:

* Storing passwords securely in environment variables.
* Enabling the front-end team to experiment with a new dark mode via a config setting.

Key tasks:

* **Products DB**: Secure the `POSTGRES_PASSWORD` environment variable.
* **Stock API**: Use a Go app that connects to the PostgreSQL database with a connection string.
* **Products API**: Update the Spring Boot application’s `application.properties` with the new password.
* **Web App**: A .NET Core app that uses `/app/secrets/api.json` for overriding settings, including API URLs and theme configurations.

Environment variables:

* `Widgetario__Theme=dark` (for dark mode)

### Solution 3: PostgreSQL Replication & Caching

Enabled local caching for the Stock API (`/cache` directory) to survive Pod restarts. We also switched to a replicated PostgreSQL setup using `widgetario/products-db:postgres-replicated`.

* **PostgreSQL Replication**: Configure the Products API to connect to the primary DB and the Stock API to the secondary.
* **Deployment Update**: Clean up old persistent volumes and services.

Commands:

```bash
kubectl delete deploy products-db
kubectl delete svc products-db
kubectl delete pvc -l app=products-db

kubectl apply -f hackathon/solution-part-3/products-db \
              -f hackathon/solution-part-3/products-api \
              -f hackathon/solution-part-3/stock-api \
              -f hackathon/solution-part-3/web

kubectl rollout restart deploy/products-api deploy/stock-api
```

### Solution 4: Ingress Configuration

Configured Ingress resources for clean domain-based routing:

* **Web App**: `http://widgetario.local`
* **API**: `http://api.widgetario.local`

Commands:

```bash
./scripts/add-to-hosts.sh widgetario.local 127.0.0.1
./scripts/add-to-hosts.sh api.widgetario.local 127.0.0.1
```

This ensures that the web app and APIs are accessible through friendly domain names.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.


