# ArgoCD + Crossplane Integration Best Practices

## Overview
This document outlines the best practices for integrating ArgoCD with Crossplane to avoid OutOfSync issues and ensure proper GitOps workflow.

## Key Issues and Solutions

### 1. Resource Tracking Method
**Issue**: ArgoCD's default label-based tracking conflicts with Crossplane's label propagation.

**Solution**: Use annotation-based tracking by setting `application.resourceTrackingMethod: annotation` in the `argocd-cm` ConfigMap.

### 2. Composite Resource Management
**Issue**: ArgoCD tries to manage Composite Resources (XR) that are dynamically created by Crossplane Claims (XRC).

**Solution**: 
- Deploy Claims (XRC) through ArgoCD
- Exclude Composite Resources (XR) from ArgoCD management
- Use the pattern: XRC → XR (managed by Crossplane) → Managed Resources

### 3. Connection Secrets Handling
**Issue**: `writeConnectionSecretsToRef` fields can cause sync drift due to dynamic updates.

**Solutions**:
- Always specify both `name` and `namespace` in `writeConnectionSecretsToRef`
- Consider using External Secrets Operator for secret management
- Add connection secrets to ArgoCD ignore differences if needed

### 4. Resource Exclusions
**Required exclusions in `argocd-cm`**:
```yaml
resource.exclusions: |
  - apiGroups: ["*"]
    kinds: ["ProviderConfigUsage"]
  - apiGroups: ["demo.io"]
    kinds: ["XCloudRunApp", "XPostgreSQL"]
```

### 5. Performance Tuning
**Issue**: Large number of CRDs can cause performance issues.

**Solution**: Increase ArgoCD Kubernetes client QPS:
```bash
export ARGOCD_K8S_CLIENT_QPS=300
```

## Sync Policy Recommendations

### For Infrastructure Applications (Compositions, XRDs):
```yaml
syncPolicy:
  automated:
    prune: false  # Never auto-prune infrastructure
    selfHeal: true
  syncOptions:
  - CreateNamespace=true
  - RespectIgnoreDifferences=true
```

### For Application Claims (XRCs):
```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
  - CreateNamespace=true
  - RespectIgnoreDifferences=true
  - ApplyOutOfSyncOnly=true
ignoreDifferences:
- group: demo.io
  kind: CloudRunApp
  jqPathExpressions:
  - .status
  - .metadata.finalizers
  - .metadata.generation
  - .metadata.resourceVersion
  - .metadata.uid
```

## Troubleshooting

### Common OutOfSync Causes:
1. **Status field changes**: Add status fields to `ignoreDifferences`
2. **Finalizer drift**: Include `.metadata.finalizers` in ignore patterns
3. **Resource version changes**: Ignore `.metadata.resourceVersion` and `.metadata.generation`
4. **Connection secret updates**: Ensure proper namespace specification

### Debugging Commands:
```bash
# Check ArgoCD application status
kubectl get applications -n argocd

# View application details
kubectl describe application demo-app -n argocd

# Check Crossplane resource status
kubectl get xr
kubectl get xrc -A

# View connection secrets
kubectl get secrets -n demo | grep connection
```

## Migration Steps

1. **Apply ArgoCD Configuration**:
   ```bash
   kubectl apply -f argocd-crossplane-config.yaml
   ```

2. **Restart ArgoCD Components**:
   ```bash
   kubectl rollout restart deployment argocd-application-controller -n argocd
   kubectl rollout restart deployment argocd-server -n argocd
   ```

3. **Refresh Applications**:
   ```bash
   argocd app sync demo-app
   argocd app sync demo-infrastructure
   ```

4. **Verify Sync Status**:
   ```bash
   argocd app list
   ```

## Expected Results

After implementing these configurations:
- ArgoCD applications should show "Synced" status
- Crossplane Claims (XRC) are managed by ArgoCD
- Composite Resources (XR) are ignored by ArgoCD
- Connection secrets are properly created and managed
- No spurious OutOfSync notifications

## Additional Resources

- [Crossplane Documentation: ArgoCD Integration](https://docs.crossplane.io/latest/guides/crossplane-with-argo-cd/)
- [ArgoCD Documentation: Resource Health](https://argo-cd.readthedocs.io/en/stable/operator-manual/health/)
- [ArgoCD Documentation: Sync Options](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/)