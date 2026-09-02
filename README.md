# Trip Planner - Kubernetes Deployment

This directory contains Kubernetes manifests for deploying the Trip Planner application to a Kubernetes cluster (Minikube for local development).

## 📁 Files Overview

| File | Purpose | Resource Type |
|------|---------|---------------|
| `secrets.yaml` | Stores sensitive data (DB password, JWT secret) | Secret |
| `postgres-pvc.yaml` | Persistent storage for PostgreSQL (2Gi) | PersistentVolumeClaim |
| `postgres-deployment.yaml` | PostgreSQL database pod | Deployment |
| `postgres-service.yaml` | Internal service for database access | Service (ClusterIP) |
| `api-deployment.yaml` | Flask API backend pod | Deployment |
| `api-service.yaml` | Internal service for API access | Service (ClusterIP) |
| `ui-deployment.yaml` | React frontend pod | Deployment |
| `ui-service.yaml` | External service for UI access | Service (NodePort) |

## 🚀 Deployment Steps

### Prerequisites

1. **Start Minikube:**
   ```bash
   minikube start --memory=2048 --cpus=2 --driver=docker
   ```

2. **Verify cluster is running:**
   ```bash
   kubectl cluster-info
   kubectl get nodes
   ```

### Deploy in Order

**Step 1: Create Secrets**
```bash
kubectl apply -f secrets.yaml
kubectl get secrets
```

**Step 2: Deploy Database Layer**
```bash
# Create persistent storage
kubectl apply -f postgres-pvc.yaml

# Deploy PostgreSQL
kubectl apply -f postgres-deployment.yaml

# Create database service
kubectl apply -f postgres-service.yaml

# Verify database is running
kubectl get pods
kubectl logs <postgres-pod-name>
```

**Step 3: Deploy API Layer**
```bash
# Deploy Flask API
kubectl apply -f api-deployment.yaml

# Create API service
kubectl apply -f api-service.yaml

# Verify API is running
kubectl get pods
kubectl logs <api-pod-name>
```

**Step 4: Deploy UI Layer**
```bash
# Deploy React UI
kubectl apply -f ui-deployment.yaml

# Create UI service
kubectl apply -f ui-service.yaml

# Verify UI is running
kubectl get pods
```

### Access the Application

**Terminal 1 - Port forward API:**
```bash
kubectl port-forward service/api 5000:5000
```

**Terminal 2 - Access UI:**
```bash
minikube service ui
```

This will open the UI in your browser. The UI will connect to the API via `http://localhost:5000` (through the port-forward).

## 🔍 Useful Commands

### View Resources
```bash
# All resources
kubectl get all

# Specific resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get pvc

# Detailed info
kubectl describe pod <pod-name>
kubectl describe service <service-name>
```

### View Logs
```bash
# View logs
kubectl logs <pod-name>

# Follow logs (live)
kubectl logs -f <pod-name>

# Last 50 lines
kubectl logs --tail=50 <pod-name>
```

### Debug Pods
```bash
# Get shell in pod
kubectl exec -it <pod-name> -- /bin/bash

# Check pod events
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Scale Deployments
```bash
# Scale API to 3 replicas
kubectl scale deployment trip-planner-api --replicas=3

# Verify scaling
kubectl get pods
```

### Update Deployments
```bash
# After changing YAML
kubectl apply -f api-deployment.yaml

# Check rollout status
kubectl rollout status deployment/trip-planner-api

# View rollout history
kubectl rollout history deployment/trip-planner-api
```

### Rollback Deployments
```bash
# Rollback to previous version
kubectl rollout undo deployment/trip-planner-api

# Rollback to specific revision
kubectl rollout undo deployment/trip-planner-api --to-revision=2
```

## 🧹 Cleanup

**Delete all resources:**
```bash
kubectl delete -f ui-service.yaml
kubectl delete -f ui-deployment.yaml
kubectl delete -f api-service.yaml
kubectl delete -f api-deployment.yaml
kubectl delete -f postgres-service.yaml
kubectl delete -f postgres-deployment.yaml
kubectl delete -f postgres-pvc.yaml
kubectl delete -f secrets.yaml
```

**Or delete everything at once:**
```bash
kubectl delete -f .
```

**Stop Minikube:**
```bash
minikube stop
```

**Delete Minikube cluster:**
```bash
minikube delete
```

## 📊 Architecture

```
┌─────────────────────────────────────┐
│        Minikube Cluster             │
│                                     │
│  ┌──────────┐  ┌──────────┐         │
│  │ UI Pod   │  │ API Pod  │         │
│  │ Port:3000│  │ Port:5000│         │
│  └────┬─────┘  └────┬─────┘         │
│       │             │               │
│  ┌────▼─────┐  ┌────▼──────┐        │
│  │UI Service│  │API Service│        │
│  │NodePort  │  │ClusterIP  │        │
│  └──────────┘  └────┬──────┘        │
│                     │               │
│                ┌────▼─────────┐     │
│                │Postgres Pod  │     │
│                │Port: 5432    │     │
│                └────┬─────────┘     │
│                     │               │
│                ┌────▼─────────┐     │
│                │Postgres Svc  │     │
│                │ClusterIP     │     │
│                └──────────────┘     │
│                     │               │
│                ┌────▼─────────┐     │
│                │ PVC (2Gi)    │     │
│                └──────────────┘     │
└─────────────────────────────────────┘
```

## 🔐 Security Notes

**Current Setup (Learning):**
- Secrets are base64 encoded (NOT encrypted)
- Passwords are simple ("password")
- No network policies

**For Production:**
- Use external secret managers (AWS Secrets Manager, Vault)
- Strong passwords with rotation
- Implement NetworkPolicies
- Enable RBAC
- Use Ingress with TLS
- Scan images for vulnerabilities

## 📚 Next Steps

1. **Learn Helm** - Convert these YAMLs to a Helm chart
2. **GitOps** - Set up ArgoCD for automated deployments
3. **Monitoring** - Add Prometheus & Grafana
4. **CI/CD** - Integrate with GitHub Actions
5. **Production** - Deploy to AWS EKS with RDS

## 🎯 Resource Limits

| Component | Memory Limit | CPU Limit | Replicas |
|-----------|-------------|-----------|----------|
| PostgreSQL | 512Mi | 500m | 1 |
| API | 256Mi | 250m | 1 |
| UI | 128Mi | 100m | 1 |
| **Total** | **896Mi** | **850m** | **3** |

Minikube allocation: 2048Mi (2Gi), so we have plenty of headroom!
