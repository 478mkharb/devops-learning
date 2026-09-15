# Kubernetes ConfigMaps, Secrets and Namespaces Interview Questions and Answers

## 1. What is a ConfigMap?
A ConfigMap stores non-confidential configuration data as key-value pairs. It separates configuration from container images.

## 2. What data belongs in a ConfigMap?
Examples include:
- Application mode
- URLs
- Feature flags
- Configuration files
- Environment-specific settings

Do not store passwords or tokens in a ConfigMap.

## 3. How can a Pod consume a ConfigMap?
A Pod can consume it through:
- Environment variables
- Individual environment variables
- Mounted files
- Command-line arguments

## 4. Create a ConfigMap using YAML.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  APP_PORT: "8080"
```

## 5. How do you create a ConfigMap imperatively?
```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_PORT=8080
```

## 6. How do you use a ConfigMap as environment variables?
```yaml
envFrom:
  - configMapRef:
      name: app-config
```

## 7. How do you use one ConfigMap key as an environment variable?
```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```

## 8. How do you mount a ConfigMap as files?
```yaml
volumes:
  - name: config-volume
    configMap:
      name: app-config

containers:
  - name: app
    image: nginx
    volumeMounts:
      - name: config-volume
        mountPath: /etc/app-config
```

## 9. What happens when a ConfigMap changes?
Mounted ConfigMap files are normally updated eventually by the kubelet. Environment variables and command arguments do not update inside an already-running container; the Pod generally must be restarted.

## 10. What is a Secret?
A Secret stores sensitive data such as passwords, tokens, keys, and certificates.

## 11. Is Kubernetes Secret data encrypted by default?
Secret values are base64-encoded in manifests and API responses, not automatically encrypted merely because they use base64. Encryption at rest must be configured in the cluster datastore, and access must be restricted with RBAC.

## 12. Create a Secret using YAML.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  DB_USER: appuser
  DB_PASSWORD: strong-password
```

## 13. What is the difference between `data` and `stringData`?
- `data` requires base64-encoded values.
- `stringData` accepts plain strings and Kubernetes converts them into Secret data.

## 14. How do you create a Secret imperatively?
```bash
kubectl create secret generic db-secret \
  --from-literal=DB_USER=appuser \
  --from-literal=DB_PASSWORD='strong-password'
```

## 15. How do you consume a Secret as environment variables?
```yaml
envFrom:
  - secretRef:
      name: db-secret
```

## 16. How do you mount a Secret as files?
```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

## 17. What are common Secret types?
- `Opaque`
- `kubernetes.io/tls`
- `kubernetes.io/dockerconfigjson`
- `kubernetes.io/basic-auth`
- `kubernetes.io/ssh-auth`
- ServiceAccount token-related types

## 18. What is the difference between ConfigMap and Secret?
| ConfigMap | Secret |
|---|---|
| Non-sensitive configuration | Sensitive configuration |
| URLs and flags | Passwords and tokens |
| Not intended for confidential data | Requires stronger access protection |
| Commonly stored as plain values | Values are encoded and may be encrypted at rest |

## 19. What is a Namespace?
A Namespace provides a logical scope for namespaced Kubernetes resources.

Examples:
- `default`
- `kube-system`
- `kube-public`
- `kube-node-lease`

## 20. Why use Namespaces?
Namespaces help with:
- Team isolation
- Environment separation
- RBAC boundaries
- Resource quotas
- Network policies
- Naming organization

## 21. Are all Kubernetes resources namespaced?
No. Pods, Deployments, Services, ConfigMaps, Secrets, and PVCs are namespaced. Nodes, PersistentVolumes, StorageClasses, Namespaces, and ClusterRoles are cluster-scoped.

## 22. How do you create and use a Namespace?
```bash
kubectl create namespace dev
kubectl get pods -n dev
kubectl config set-context --current --namespace=dev
```

## 23. What is ResourceQuota?
ResourceQuota limits aggregate resource consumption in a Namespace.

It can limit:
- CPU
- Memory
- Number of Pods
- Number of Services
- Number of Secrets
- PVC storage

## 24. What is LimitRange?
LimitRange defines default, minimum, and maximum resource requests or limits for objects in a Namespace.

## 25. Can a Pod access a Secret in another Namespace?
No. Secrets are namespace-scoped. The Secret must exist in the same Namespace as the Pod.

## 26. How do you troubleshoot a missing ConfigMap or Secret?
```bash
kubectl get configmap -n <namespace>
kubectl get secret -n <namespace>
kubectl describe pod <pod-name>
kubectl describe configmap <name>
kubectl describe secret <name>
```

Check the Namespace, object name, key name, RBAC permissions, and Pod events.

## 27. What is the security best practice for Secrets?
- Use least-privilege RBAC.
- Enable encryption at rest.
- Avoid committing Secrets to Git.
- Use external secret managers when appropriate.
- Restrict Namespace access.
- Rotate credentials.
- Avoid exposing Secret values in logs.
- Do not treat base64 as encryption.

## 28. Explain ConfigMaps, Secrets and Namespaces in an interview.
ConfigMaps store non-sensitive configuration, Secrets store sensitive values, and Namespaces provide logical and administrative isolation for namespaced resources. ConfigMaps and Secrets can be injected as environment variables or mounted as files. Namespaces are commonly combined with RBAC, ResourceQuota, LimitRange, and NetworkPolicy to separate teams and environments.
