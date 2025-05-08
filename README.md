# kubernetes_hackathon
This project demonstrates a containerized microservice architecture deployed on a local Kubernetes cluster (Minikube). It's intended for use during a hackathon, running entirely inside a VirtualBox VM using Ubuntu.

** Requirements**
- VirtualBox installed on your host machine
- Ubuntu (20.04 or later) installed in VirtualBox(in our case we used server)
- Internet connection in your Ubuntu VM
**
 Inside Ubuntu VM:**
- Docker
- Minikube
- Kubectl
- Git

**** Setup Instructions****

**1. Update System & Install Dependencies** 
sudo apt update && sudo apt upgrade -y

**2. Install Docker**
sudo apt install -y docker.io
sudo usermod -aG docker $USER
newgrp docker

**3. Install Minikube**
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

** 4. Install Kubectl**
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

** 5. Start Minikube**
minikube start --driver=docker

 Note: If using VirtualBox driver, replace `--driver=docker` with `--driver=virtualbox`

** 6. Clone the Project Repository**
git clone https://github.com/yourusername/your-repo.git
cd your-repo

**
 7. Deploy to Kubernetes**
kubectl apply -f k8s/


** 8. Access the Application**

Use the following to get the service URL:


solution 1
Widgetario is a company which sells gadgets and they want to run their public web app on Kubernetes.  
All the components are packaged into container images and published on Docker Hub.

The component names in the diagram are the DNS names the app expects to use. And when you're working on the YAML, we decided to start with one replica for one component.
 we were able to ping  (127.0.0.1) or browse port 8080 on cluster

 solution 2

If we `docker image inspect widgetario/products-db:21.03` we will see the database password is in the default environment variables, so if someone manages to get the image they'll know our production password.

Also the front-end team are experimenting with a new dark mode, and they want to quickly turn it on and off with a config setting
