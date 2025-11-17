# Kubernetes Deployment Guide

This directory contains Kubernetes manifest files for deploying the Node.js application to a Kubernetes cluster.

## Prerequisites

- Kubernetes cluster (local or cloud-based)
- `kubectl` CLI tool installed and configured
- Docker image built and available to the cluster

## Files Overview

- **namespace.yaml** - Creates a dedicated namespace for the application
- **configmap.yaml** - Stores configuration data (environment variables)
- **deployment.yaml** - Defines the application deployment with 3 replicas
- **service.yaml** - Exposes the application via LoadBalancer
- **ingress.yaml** - (Optional) Provides external HTTP/HTTPS access
- **hpa.yaml** - (Optional) Horizontal Pod Autoscaler for automatic scaling

## Quick Start

### 1. Build and Tag Docker Image

```bash
# Build the Docker image
docker build -t node-project:latest .

# If using a registry, tag and push
docker tag node-project:latest <your-registry>/node-project:latest
docker push <your-registry>/node-project:latest
```

### 2. Deploy to Kubernetes

Deploy all resources in order:

```bash
# Create namespace
kubectl apply -f deploy/namespace.yaml

# Create ConfigMap
kubectl apply -f deploy/configmap.yaml

# Create Deployment
kubectl apply -f deploy/deployment.yaml

# Create Service
kubectl apply -f deploy/service.yaml

# (Optional) Create Ingress
kubectl apply -f deploy/ingress.yaml

# (Optional) Create HPA
kubectl apply -f deploy/hpa.yaml
```

Or deploy everything at once:

```bash
kubectl apply -f deploy/
```

### 3. Verify Deployment

```bash
# Check namespace
kubectl get namespaces

# Check all resources in the namespace
kubectl get all -n node-project

# Check pods status
kubectl get pods -n node-project

# Check service
kubectl get svc -n node-project

# View logs
kubectl logs -n node-project -l app=node-project

# Describe deployment
kubectl describe deployment node-project -n node-project
```

### 4. Access the Application

#### Using LoadBalancer Service

```bash
# Get the external IP
kubectl get svc node-project-service -n node-project

# Wait for EXTERNAL-IP to be assigned (may take a few minutes)
# Access the app at http://<EXTERNAL-IP>
```

#### Using Port Forwarding (for testing)

```bash
kubectl port-forward -n node-project svc/node-project-service 3000:80

# Access at http://localhost:3000
```

#### Using Ingress

If you deployed the Ingress resource:

1. Ensure you have an Ingress controller installed (e.g., nginx-ingress)
2. Add entry to `/etc/hosts`: `<ingress-ip> node-project.local`
3. Access at http://node-project.local

## Configuration

### Updating Environment Variables

Edit `configmap.yaml` and reapply:

```bash
kubectl apply -f deploy/configmap.yaml
kubectl rollout restart deployment/node-project -n node-project
```

### Scaling

#### Manual Scaling

```bash
kubectl scale deployment node-project -n node-project --replicas=5
```

#### Automatic Scaling (HPA)

The HPA is configured to scale between 2-10 replicas based on CPU and memory usage.

```bash
# Check HPA status
kubectl get hpa -n node-project

# Describe HPA
kubectl describe hpa node-project-hpa -n node-project
```

## Updating the Application

```bash
# Build new image with a version tag
docker build -t node-project:v1.1.0 .

# Update the deployment
kubectl set image deployment/node-project node-project=node-project:v1.1.0 -n node-project

# Check rollout status
kubectl rollout status deployment/node-project -n node-project

# Rollback if needed
kubectl rollout undo deployment/node-project -n node-project
```

## Troubleshooting

### Pods not starting

```bash
# Check pod status
kubectl get pods -n node-project

# View pod details
kubectl describe pod <pod-name> -n node-project

# Check logs
kubectl logs <pod-name> -n node-project

# Check events
kubectl get events -n node-project --sort-by='.lastTimestamp'
```

### Service not accessible

```bash
# Check service endpoints
kubectl get endpoints -n node-project

# Test from within cluster
kubectl run -it --rm debug --image=busybox --restart=Never -n node-project -- wget -O- http://node-project-service
```

### Image pull errors

If using a private registry:

```bash
# Create a secret for registry authentication
kubectl create secret docker-registry regcred \
  --docker-server=<your-registry> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email> \
  -n node-project

# Add to deployment.yaml under spec.template.spec:
# imagePullSecrets:
# - name: regcred
```

## Clean Up

Remove all resources:

```bash
# Delete all resources in the namespace
kubectl delete namespace node-project

# Or delete individually
kubectl delete -f deploy/
```

## Production Considerations

1. **Image Registry**: Use a proper container registry (Docker Hub, ECR, GCR, etc.)
2. **Image Tags**: Use specific version tags instead of `latest`
3. **Resource Limits**: Adjust CPU/memory limits based on your application needs
4. **Health Checks**: Customize liveness and readiness probes for your endpoints
5. **Secrets**: Use Kubernetes Secrets for sensitive data
6. **TLS/SSL**: Enable HTTPS using cert-manager or cloud provider certificates
7. **Monitoring**: Set up Prometheus and Grafana for monitoring
8. **Logging**: Configure centralized logging (ELK stack, Loki, etc.)
9. **Backup**: Implement backup strategies for persistent data
10. **Security**: Apply Pod Security Standards and Network Policies

## Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)

