# Two-Tier Flask + MySQL Application on Kubernetes

A two-tier web application deployed on a Kubernetes (KIND) cluster, demonstrating containerization, orchestration, and DevOps deployment practices. The application consists of a Python Flask frontend/backend service connected to a MySQL database, both running as separate deployments and exposed via Kubernetes Services.

## Architecture

```
                        ┌─────────────────────────┐
   Browser  ── :30036 ─▶│   flask-service          │
                        │   (NodePort)             │
                        └───────────┬──────────────┘
                                    │
                        ┌───────────▼──────────────┐
                        │   flask-deployment        │
                        │   (2 replicas)            │
                        └───────────┬──────────────┘
                                    │
                        ┌───────────▼──────────────┐
                        │   mysql-service            │
                        │   (ClusterIP)              │
                        └───────────┬──────────────┘
                                    │
                        ┌───────────▼──────────────┐
                        │   mysql-deployment         │
                        │   (1 replica)              │
                        └───────────────────────────┘
```

## Tech Stack

- **Frontend/Backend:** Python Flask
- **Database:** MySQL 8.0
- **Containerization:** Docker
- **Orchestration:** Kubernetes (KIND cluster)
- **Hosting:** AWS EC2
- **Image Registry:** DockerHub

## Repository Structure

```
two-tier-app-k8s/
├── app.py                     # Flask application
├── requirements.txt           # Python dependencies
├── Dockerfile                 # Container build instructions
├── message.sql                # (optional) manual DB seed script
├── K8s-manifests/
│   ├── flask-deployment.yaml
│   ├── flask-service.yaml
│   ├── mysql-deployment.yaml
│   └── mysql-service.yaml
├── kind-config.yaml            # KIND cluster configuration
└── README.md
```

## Prerequisites

- Docker
- kubectl
- KIND (`kind`)
- An AWS EC2 instance (or any Linux host) with Docker installed

## Setup & Deployment

### 1. Clone the repository

```bash
git clone https://github.com/Ranawaqas323421/two-tier-app-k8s.git
cd two-tier-app-k8s
```

### 2. Build and push the Docker image

```bash
docker build -t waqas323421/two-tier-app:v1 .
docker login
docker push waqas323421/two-tier-app:v1
```

### 3. Create the KIND cluster

```bash
kind create cluster --config kind-config.yaml
```

### 4. Create the MySQL secret

```bash
kubectl create secret generic mysql-secret \
  --from-literal=MYSQL_ROOT_PASSWORD=<your-password> \
  --from-literal=MYSQL_USER=<your-user> \
  --from-literal=MYSQL_PASSWORD=<your-password>
```

### 5. Apply the Kubernetes manifests

```bash
cd K8s-manifests
kubectl apply -f .
```

### 6. Verify the deployment

```bash
kubectl get pods
kubectl get svc
```

Both `flask-deployment` and `mysql-deployment` pods should show `1/1 Running`.

### 7. Access the application

```
http://<EC2-public-ip>:30036
```

Make sure port `30036` is open in your EC2 security group's inbound rules.

## Useful Commands

| Purpose                          | Command                                                    |
|-----------------------------------|-------------------------------------------------------------|
| View pod logs                     | `kubectl logs <pod-name>`                                   |
| Describe a pod (debug)            | `kubectl describe pod <pod-name>`                            |
| Restart a deployment              | `kubectl rollout restart deployment flask-deployment`        |
| Update image on a running deploy  | `kubectl set image deployment/flask-deployment flask-app=<new-image>` |
| Port-forward for local testing    | `kubectl port-forward svc/flask-service 5000:5000 --address 0.0.0.0` |
| Delete the cluster                | `kind delete cluster --name two-tier-cluster`                |

## Troubleshooting

- **ImagePullBackOff** — Verify the image name/tag in `flask-deployment.yaml` matches what was pushed to DockerHub.
- **CreateContainerConfigError** — Usually means the `mysql-secret` hasn't been created yet.
- **App not reachable externally** — Check the EC2 security group allows inbound traffic on the NodePort, and confirm the KIND `extraPortMappings` in `kind-config.yaml` match the service's NodePort.

## Author

**Waqas Saleem**
GitHub: [Ranawaqas323421](https://github.com/Ranawaqas323421)
