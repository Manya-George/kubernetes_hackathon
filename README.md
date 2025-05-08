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
* Products DB is the simplest - it just needs a password stored and surfaced in the `POSTGRES_PASSWORD` environment variable in the Pod (of course - anything with a password is sensitive data).

* The Stock API is a Go app. It uses an environment variable for the database connection string, and the password will need to match the DB. The team thinks the environment variable name starts with `POSTGRES`.

* The Products API is a Java app. The team who built it left to found a startup and we have no documentation. It's a Spring Boot app though, so the config files are usually called `application.properties` and we'll need to update the password there too.

* The web app is .NET Core. We know quite a lot about this - it reads default config from `/app/appsettings.json` but it will override any settings it finds in `/app/secrets/api.json`. We want to update the URLs for the APIs to use fully-qualified domain names.

* That feature flag for the UI can be set with an environment variable - `Widgetario__Theme` = `dark`.
  
**  solution 3**

Enabled local caching for the Stock API (/cache directory) to survive Pod restarts without needing persistent storage.

Switched to a replicated PostgreSQL setup using widgetario/products-db:postgres-replicated.

Configured the Products API to connect to the primary DB, and the Stock API to connect to the secondary.

Updated deployments and cleaned up old persistent volumes and services.Below is the code:

kubectl delete deploy products-db
kubectl delete svc products-db
kubectl delete pvc -l app=products-db

kubectl apply -f hackathon/solution-part-3/products-db \
              -f hackathon/solution-part-3/products-api \
              -f hackathon/solution-part-3/stock-api \
              -f hackathon/solution-part-3/web

kubectl rollout restart deploy/products-api deploy/stock-api

**solution 4**
Configured Ingress resources for clean domain-based routing:

Web app: http://widgetario.local

API: http://api.widgetario.local

Deployed an ingress controller and ensured the services were accessible via standard ports.Below is the code :

./scripts/add-to-hosts.sh widgetario.local 127.0.0.1
./scripts/add-to-hosts.sh api.widgetario.local 127.0.0.1

