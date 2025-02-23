# Client API Service

## Overview
This project implements a client API service with MongoDB integration, deployed using Docker and Kubernetes with SSL/TLS support via cert-manager.

## Prerequisites
- Docker
- Kubernetes cluster
- kubectl CLI
- Python 3.10+
- Jenkins (for CI/CD)

## Project Structure
```
.
├── client_api/
│   ├── Dockerfile
│   ├── main.py
│   ├── requirements.txt
│   └── debug_mongo.sh
├── manifests/
│   ├── cert-manager.yaml
│   ├── certificate.yaml
│   ├── client_api.yaml
│   ├── cluster-issuer.yaml
│   ├── ingress.yaml
│   └── mongodb.yaml
├── Jenkinsfile
└── Documentation.pdf
```

## CI/CD Pipeline
The project uses Jenkins for continuous integration and deployment with the following stages:
1. **Checkout Code**: Checkout SCM from repository
2. **Build Docker Image**: Build image with tag
3. **Push Docker Image**: Push image to Docker Hub
4. **Update Kubernetes Manifests**: Update image tag in YAML
5. **Deploy to Kubernetes**: Apply manifests to cluster
6. **Build Acceptance Test**: Run BAT tests
7. **Post Actions**: Handle Success/Failure/Always conditions

## API Endpoints

### Root Endpoint
- **URL**: `/`
- **Method**: GET
- **Response**: `{"message": "Welcome to Client APIs"}`

### Health Check
- **URL**: `/health`
- **Method**: GET
- **Response**: Success (200)

## Deployment

### Prerequisites
1. Kubernetes cluster is running
2. kubectl is configured with correct context
3. Docker registry credentials are configured

### Kubernetes Components

| Component | File Name | Purpose | Configuration Details | Dependencies |
|-----------|-----------|---------|----------------------|--------------|
| Cert-Manager | cert-manager.yaml | Certificate Manager | None | None |
| ClusterIssuer | cluster-issuer.yaml | Certificate Issuer | - Type: Let's Encrypt Staging<br>- Email: dinesh.pundkar@gmail.com<br>- Challenge: HTTP01 | Cert-Manager |
| Certificate | certificate.yaml | SSL/TLS Certificate | - Domains: deltacapita.com, *.deltacapita.com<br>- Secret: dsp-capita-cert-tls | ClusterIssuer |
| Ingress | ingress.yaml | Traffic Routing | - Host: clients.api.deltacapita.com<br>- TLS Enabled | Nginx Ingress, Certificate |
| MongoDB | mongodb.yaml | Database | - Port: 27017<br>- Single Standalone instance mode<br>- No persistent storage configured | None |
| dsp-api | client-api.yaml | API | - Client API pods & Service | Client API |

### Deployment Steps
1. Install Nginx Ingress Controller:
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
   ```

2. Install cert-manager:
   ```bash
   kubectl apply -f manifests/cert-manager.yaml
   ```

3. Configure SSL/TLS:
   ```bash
   kubectl apply -f manifests/cluster-issuer.yaml
   kubectl apply -f manifests/certificate.yaml
   ```

4. Deploy MongoDB and API:
   ```bash
   kubectl apply -f manifests/mongodb.yaml
   kubectl apply -f manifests/client_api.yaml
   ```

5. Configure Ingress:
   ```bash
   kubectl apply -f manifests/ingress.yaml
   ```

### Verifying Deployment
```bash
# Check pods status
kubectl get pods

# Check services
kubectl get svc

# Check ingress
kubectl get ingress

# Check certificates
kubectl get certificates
```

## Future Improvements

### Infrastructure & Scalability
- Implement MongoDB replica set for high availability
- Configure Persistent Volumes (PV/PVC) for MongoDB data persistence
- Set up Horizontal Pod Autoscaling (HPA)
- Implement pod disruption budgets

### Monitoring & Observability
- Deploy Prometheus/Grafana stack
- Set up ELK stack for centralized logging
- Create custom dashboards
- Implement automated alerting

### Security Enhancements
- Set up Vault for secrets management
- Enable MongoDB authentication and encryption
- Regular security scanning using Trivy

### Development & CI/CD
- Set up GitOps workflow using ArgoCD or Flux
- Implement canary deployments
- Implement feature flags

### Backup & Recovery
- Implement automated MongoDB backup solution
- Implement backup for K8s cluster using Velero
- Implement automated backup testing
- Set up disaster recovery procedures

### Compliance & Governance
- Implement audit logging
- Add data retention and archival policies
