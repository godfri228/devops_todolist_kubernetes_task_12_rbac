# RBAC Task Validation Instructions

## Prerequisites
- Docker installed and running
- kubectl installed
- kind installed

## Step-by-Step Validation

### 1. Create Kind Cluster
```bash
kind create cluster --config cluster.yml
```

### 2. Apply RBAC Configuration
```bash
kubectl apply -f security/rbac.yml
```

### 3. Deploy Application
```bash
kubectl apply -f deployment.yml
```

### 4. Wait for Pod to be Ready
```bash
kubectl wait --for=condition=ready pod -l app=secret-reader --timeout=60s
```

### 5. Get Pod Name
```bash
POD_NAME=$(kubectl get pods -l app=secret-reader -o jsonpath='{.items[0].metadata.name}')
echo "Pod name: $POD_NAME"
```

### 6. Execute Curl Command to List Secrets
```bash
kubectl exec -it $POD_NAME -- sh -c '
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
APISERVER=https://kubernetes.default.svc
curl -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
     $APISERVER/api/v1/namespaces/default/secrets
'
```

### 7. Alternative Method (if curl formatting is needed)
```bash
kubectl exec -it $POD_NAME -- sh -c '
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
APISERVER=https://kubernetes.default.svc
curl -s -H "Authorization: Bearer $TOKEN" \
     --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
     $APISERVER/api/v1/namespaces/default/secrets | head -20
'
```

## Expected Output
The curl command should return a JSON response containing the list of secrets in the default namespace. The response should include:
- `kind: SecretList`
- `apiVersion: v1`
- `items: []` (array of secrets)

## Verification Steps
1. Verify ServiceAccount was created:
   ```bash
   kubectl get serviceaccount secret-reader
   ```

2. Verify Role was created:
   ```bash
   kubectl get role secret-reader-role
   ```

3. Verify RoleBinding was created:
   ```bash
   kubectl get rolebinding secret-reader-binding
   ```

4. Verify Deployment uses correct ServiceAccount:
   ```bash
   kubectl get deployment secret-reader-deployment -o yaml | grep serviceAccountName
   ```

## Cleanup
```bash
kind delete cluster --name rbac-test-cluster
```

## Troubleshooting
- If pod is not ready, check logs: `kubectl logs -l app=secret-reader`
- If RBAC permission denied, verify role and binding are correctly applied
- If curl fails, ensure the pod has the correct service account token mounted