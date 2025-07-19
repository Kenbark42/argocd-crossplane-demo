# ArgoCD + Crossplane GitOps Demo

This repository demonstrates GitOps principles using ArgoCD and Crossplane to manage cloud infrastructure.

## 🏗️ Architecture

- **ArgoCD**: GitOps continuous delivery
- **Crossplane**: Cloud infrastructure management
- **Cloud Run**: Serverless container hosting
- **Cloud SQL**: Managed PostgreSQL database

## 📁 Repository Structure

```
├── applications/     # ArgoCD Application definitions
├── infrastructure/   # Crossplane Compositions
├── manifests/       # Kubernetes manifests and resource claims
└── README.md
```

## 🚀 Deployment

1. Install ArgoCD applications:
   ```bash
   kubectl apply -f applications/
   ```

2. Infrastructure will be automatically deployed via GitOps!

## 🧪 Testing

Check the deployment:
```bash
kubectl get applications -n argocd
kubectl get postgresql,cloudrunapp -n demo
```

## 🔄 GitOps Workflow

1. Make changes to this repository
2. Commit and push to main branch
3. ArgoCD automatically syncs changes
4. Infrastructure is updated declaratively
