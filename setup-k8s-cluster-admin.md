# Create cluster admin

```
kubectl -n kube-system create serviceaccount <service-account-name>
```

```
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: <service-account-name>
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: <service-account-name>
  namespace: kube-system
EOF
```

## Fetch server url
```
kubectl config view --minify -o 'jsonpath={.clusters[0].cluster.server}'
```

```
kubectl get serviceAccounts <service-account-name> -n <namespace> -o 'jsonpath={.secrets[*].name}'
```

```
kubectl get secret <service-account-secret-name> -n <namespace> -o yaml
```