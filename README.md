# ArgoCD Nginx Deployment Demo

This repository contains a simple Nginx deployment that can be managed using ArgoCD. It demonstrates GitOps principles by managing Kubernetes resources through Git.

## Required Changes for New Users

Before deploying this application, you need to modify the following:

1. In `application.yaml`:
   ```yaml
   spec:
     source:
       repoURL: https://github.com/modhugu/argo-CD.git  # Change this to your repository URL
   ```
   - Replace the `repoURL` with your own Git repository URL
   - If using a different branch, update `targetRevision` (default is "main")
   - If needed, modify the `namespace` under `metadata` (default is "argocd")
   - If needed, modify the destination `namespace` (default is "default")

2. In `manifests/nginx/deployment.yaml`:
   - Modify resource limits/requests if needed (default: memory="128Mi", cpu="500m")
   - Change the number of replicas (default: 3)
   - Update the container image if needed (default: httpd:2)

3. In `manifests/nginx/svc.yaml`:
   - Change service type if needed (default: LoadBalancer)
   - Modify port mappings if needed (default: port 80)

## Repository Structure

```
├── application.yaml          # ArgoCD Application manifest
└── manifests
    └── nginx
        ├── deployment.yaml   # Nginx deployment configuration
        └── svc.yaml         # Service configuration for Nginx
```

## Prerequisites

- A Kubernetes cluster
- [ArgoCD installed](https://argo-cd.readthedocs.io/en/stable/getting_started/) in your cluster
- `kubectl` configured to access your cluster
- Access to ArgoCD UI (optional but recommended)

## Deployment Steps

### 1. Install ArgoCD (if not already installed)

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Access ArgoCD UI (Optional)

```bash
# Port forward the ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get the initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Then access the UI at: http://localhost:8080
- Username: admin
- Password: (use the password obtained from the previous command)

### 3. Deploy the Application

Apply the ArgoCD Application manifest:

```bash
kubectl apply -f application.yaml -n argocd
```

### 4. Verify the Deployment

```bash
# Check the ArgoCD application status
kubectl get applications -n argocd

# Check the deployed resources
kubectl get deployment,svc -n default
```

## Application Details

The deployment consists of:
- A Deployment running Apache HTTP Server (httpd:2)
  - 3 replicas
  - Resource limits and requests configured
  - Exposed on port 80
- A LoadBalancer Service
  - Exposes the deployment to external traffic
  - Maps port 80

## Sync Policy

The application is configured with automatic sync enabled:
- Auto-pruning of removed resources
- Self-healing of manual cluster changes
- Automatic namespace creation

## Troubleshooting

If the application isn't syncing properly:

1. Check Application status:
```bash
kubectl describe application nginx-application -n argocd
```

2. View ArgoCD logs:
```bash
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
```

3. Verify the repository is accessible to ArgoCD

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a new Pull Request