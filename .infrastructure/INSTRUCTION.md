RBAC Task Validation Instructions
Prerequisites

Docker installed and running
kubectl installed
kind installed

Step-by-Step Validation
1. Create Kind Cluster
bashkind create cluster --config cluster.yml
2. Apply RBAC Configuration
bashkubectl apply -f security/rbac
3. Deploy Application
bashkubectl apply -f deployment.yml
4. Wait for Pod to be Ready
bashkubectl wait --for=condition=ready pod -l app=secret-reader --timeout=60s
5. Get Pod Name
bashPOD_NAME=$(kubectl get pods -l app=secret-reader -o jsonpath='{.items[0].metadata.name}')
6. Execute Curl Command to List Secrets
bashkubectl exec -it $POD_NAME -- sh -c 'TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token); curl -H "Authorization: Bearer $TOKEN" --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt https://kubernetes.default.svc/api/v1/namespaces/default/secrets'
Expected Output
The curl command should return a JSON response with secrets list.
Cleanup
bashkind delete cluster --name rbac-test-cluster