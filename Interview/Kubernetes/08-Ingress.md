# Kubernetes Ingress Interview Questions and Answers

## 1. What is Ingress?
Ingress is a Kubernetes API resource that defines HTTP and HTTPS routing from outside the cluster to Services inside the cluster.

## 2. Does Ingress itself route traffic?
No. Ingress is a configuration object. An Ingress Controller implements the routing behavior.

## 3. What is an Ingress Controller?
An Ingress Controller watches Ingress resources and configures a reverse proxy or load balancer.

Examples:
- NGINX Ingress Controller
- Traefik
- HAProxy
- Kong
- Cloud-provider controllers

## 4. What is the basic traffic flow?
```text
Client
  |
DNS
  |
External Load Balancer
  |
Ingress Controller
  |
Kubernetes Service
  |
Pod
```

## 5. What is the difference between Service and Ingress?
| Service | Ingress |
|---|---|
| Provides stable access to Pods | Provides HTTP/HTTPS routing |
| Can expose TCP/UDP depending on type | Primarily HTTP/HTTPS |
| ClusterIP, NodePort, LoadBalancer | Host/path-based routing |
| Selects backend Pods | Selects Services |

## 6. Example Ingress.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```

## 7. What is `IngressClass`?
IngressClass identifies which controller should implement an Ingress resource.

## 8. What are Ingress path types?
- `Prefix`
- `Exact`
- `ImplementationSpecific`

`Prefix` matches URL path prefixes. `Exact` matches the complete path. `ImplementationSpecific` depends on the controller.

## 9. What is host-based routing?
Different hostnames route to different Services.

Example:
- `api.example.com` → API Service
- `app.example.com` → Frontend Service

## 10. What is path-based routing?
Different URL paths route to different Services.

Example:
- `/api` → API Service
- `/` → Frontend Service

## 11. What is TLS termination?
TLS termination means the Ingress Controller decrypts HTTPS traffic and forwards traffic to the backend according to its configuration.

## 12. Example TLS Ingress.
```yaml
spec:
  tls:
    - hosts:
        - example.com
      secretName: example-tls
```

The TLS Secret must contain the certificate and private key.

## 13. What is the difference between Ingress and Gateway API?
Ingress is a simpler, older HTTP routing API. Gateway API provides more expressive and role-oriented resources such as GatewayClass, Gateway, and HTTPRoute.

## 14. Can Ingress expose databases?
Ingress is designed mainly for HTTP/HTTPS. Databases usually use Services, internal load balancers, or protocol-specific gateways.

## 15. What happens if no Ingress Controller is installed?
Creating an Ingress object alone does not provide routing. The resource remains ineffective until a compatible controller processes it.

## 16. How do you troubleshoot Ingress?
```bash
kubectl get ingress
kubectl describe ingress <name>
kubectl get ingressclass
kubectl get pods -n <controller-namespace>
kubectl logs -n <controller-namespace> <controller-pod>
kubectl get svc
kubectl get endpointslices
```

Check DNS, LoadBalancer address, controller logs, IngressClass, TLS Secret, Service name, Service port, and backend readiness.

## 17. What causes 404 from Ingress?
- Host header does not match.
- Path does not match.
- Wrong path type.
- Controller default backend handles the request.
- Rewrite configuration is incorrect.

## 18. What causes 502 or 503 from Ingress?
- Backend Service has no ready endpoints.
- Wrong Service port.
- Application is not listening.
- Network policy blocks traffic.
- Controller cannot connect to the backend.

## 19. What is a default backend?
A default backend handles requests that do not match configured host/path rules, depending on the controller.

## 20. What are Ingress security best practices?
- Use TLS.
- Restrict exposed hosts and paths.
- Apply authentication and authorization.
- Configure rate limiting where supported.
- Keep controller updated.
- Use NetworkPolicies.
- Avoid exposing administrative endpoints.
- Monitor access logs and errors.
- Use secure headers and appropriate timeouts.
