
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
To clone our repo
    - Open a terminal (PowerShell on Windows; any shell on Linux/macOS) 
    - Run: `git clone https://github.com/kubernetes_hackathon.git`
  

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



## Part 5 - Productionizing
Deployment:

```
kubectl apply -f kubernetes_hackathon/solution-part-5/ingress-controller -f kubernetes_hackathon/solution-part-5/products-db -f kubernetes_hackathon/solution-part-5/products-api  -f kubernetes_hackathon/solution-part-5/stock-api -f kubernetes_hackathon/solution-part-5/web
```

Everything works in the same way:

> Browse to http://widgetario.local to see the app

> See the API response at http://api.widgetario.local/products



The StatefulSet rollout takes a few minutes, and the app may not be responsive until both Pods are up. **And** there are CPU resources in the specs, so if your cluster doesn't have enough capacity you may see Pods stuck in the _Pending_ status, so you'll need to adjust the values.


## Part 6 - Observability


All the API and web servers publish metrics, so we don't need to change any code. To prove monitoring is usable you'll need to:

- run a monitoring stack with Prometheus and Grafana
- configure the Pod specs so Prometheus collects metrics - the database doesn't expose any metric but the other components do
- the app devs have said the Products API using a custom metrics path `/actuator/prometheus`, and the web app might be running with a custom port
- deploy the changes and check all the components are having metrics stored in Prometheus
- open Grafana and load the dashboard from `kubernetes_hackathon/files/grafana-dashboard.json`; use the app and confirm all the visualizations show data.

Here's the dashboard you should see:

![](/img/hackathon-grafana.png)

And then you'll also need to set up centralized logging:

- run the EFK stack to collect and store logs in the app namespace
- the dev team say the web app writes logs to a file, so you'll need to add a sidecar container to print logs from `/logs/app.log` in the app container
- open Kibana and load the dashboard from `kubernetes_hackathon/files/kibana-dashboard.ndjson`; check the visualizations to see every component is writing logs

Here's what the Kibana dashboard should look like:

![](/img/hackathon-kibana.png)

<details>
  <summary>Hints</summary>

The monitoring and logging stacks are standard components, so you can run them from the specs we used in earlier labs. 

You won't need to tweak the Prometheus or Fluent Bit configuration, unless you're using a custom namespace for your Widgetario Pods...

</details><br />

The app will still look the same. You should see those fancy dashboards and be able to search for logs for each component. 

<details>
  <summary>Solution</summary>

If you didn't get part 6 finished, you can check out the specs in the sample solution from `hackathon/solution-part-6`. The specs in the `widgetario` folder have been re-organised to have one YAML files for each component.

Deploy the sample solution and you can continue to part 7:

```
kubectl apply -f hackathon/solution-part-6/monitoring -f hackathon/solution-part-6/logging -f hackathon/solution-part-6/ingress-controller -f hackathon/solution-part-6/widgetario
```

There's a change to the StatefulSet spec (to explicitly opt out of metrics collection), and it will take a while for the rollout to complete.

You can browse to the UIs using NodePort or LoadBalancer Services:

```
kubectl get svc -A -l kubernetes.courselabs.co=hackathon
```
* Grafana on http://localhost:30003 or http://localhost:3000

* Kibana on http://localhost:30005 or http://localhost:5601

</details><br/>

## Part 7 - CI/CD

The last thing we need is a full CI/CD pipeline to build from source code and deploy to a test environment.

We've settled on Helm for packaging, so the first task is to work up the YAML specs into a Helm chart. The chart should support:

- multiple release in the same namespace
- variables for image tags, replica counts, and whether resource limits should be included
- app configuration per release, including the database password and the web theme

> We'll be using standard deployments for the ingress controller, monitoring and logging stacks so you don't need to build Helm charts for those

Deploy a second instance of the Widgetario app in your cluster using Helm, with the values file [hackathon/files/helm/uat.yaml](./files/helm/uat.yaml). Confirm you can access it on a different port from the original, and all the Pods are talking to the right components.

Then we need to put together the continuous integration pipeline:

- we'll use Jenkins, BuildKit and Gogs to power the pipeline
- you'll need to create a registry Secret to push to Docker Hub (or your own registry)
- the source code is all in the `/hackathon/project` folder, which is where you'll find the [Jenkinsfile](./project/Jenkinsfile) with the build steps already in it

You should be able to add your local Gogs server as a new Git remote and push this repository to it. Then your Jenkins build should build images for the database, APIs and web server and push them all to your container registry, with a version number in the tag.

So then we're ready to add continuous deployment:

- extend the Jenkinsfile to add the deployment steps
- use the Helm chart for deploying to a test namespace
- deploy the chart with the values file [hackathon/files/helm/smoke-test.yaml](./files/helm/smoke-test.yaml)
- the deployment needs to use the latest images which were pushed in the build.

Now when you trigger the build from Jenkins you should see a new version of the app deployed in your test namespace, running all the latest image versions.

<details>
  <summary>Hints</summary>

If you've got this far, you probably don't need any hints :) The infrastructure stack we want to run is the same one we used in an earlier lab, so you can use those specs as the basis.

The default Jenkins setup from that lab creates a project which points to a different Jenkinsfile - you'll need to edit the path in the job.

You may be running low on resources, so you can scale down the existing deployment:

```
kubectl scale deploy/products-api deploy/stock-api deploy/web sts/products-db --replicas 0
```

And remember the registry secret needs to contain your own credentials, and the image name you build needs to use a repository which you have permission to push to.

</details><br />

<details>
  <summary>Solution</summary>


## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.


