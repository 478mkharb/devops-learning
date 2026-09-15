# Kubernetes Probes and Troubleshooting Interview Questions and Answers

## 1. What are Kubernetes probes?
Probes are health checks performed by the kubelet to determine container health and readiness.

The main probes are:
- Liveness probe
- Readiness probe
- Startup probe

## 2. What is a liveness probe?
A liveness probe determines whether a container is still functioning. If it repeatedly fails, kubelet restarts the container.

## 3. What is a readiness probe?
A readiness probe determines whether a Pod should receive traffic through Services. A failed readiness probe removes the Pod from matching Service endpoints, but does not necessarily restart the container.

## 4. What is a startup probe?
A startup probe gives slow-starting applications time to initialize. Until it succeeds, liveness and readiness probes are not normally executed.

## 5. Difference between liveness and readiness?
| Liveness | Readiness |
|---|---|
| Detects broken container | Detects ability to receive traffic |
| Failure can restart container | Failure removes Pod from Service endpoints |
| Protects against deadlocks | Protects users from unhealthy instances |

## 6. What probe mechanisms are available?
- HTTP GET
- TCP socket
- Exec command
- gRPC probe in supported Kubernetes versions

## 7. Example HTTP readiness probe.
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

## 8. Example liveness probe.
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  periodSeconds: 10
  failureThreshold: 3
```

## 9. Example startup probe.
```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  periodSeconds: 10
  failureThreshold: 30
```

## 10. What are important probe fields?
- `initialDelaySeconds`
- `periodSeconds`
- `timeoutSeconds`
- `successThreshold`
- `failureThreshold`
- `terminationGracePeriodSeconds`

## 11. What is `CrashLoopBackOff`?
It means a container is repeatedly starting and failing, so Kubernetes increases the delay between restart attempts.

Common causes:
- Application crash
- Wrong command
- Missing configuration
- Missing Secret
- Permission issue
- Dependency failure
- Incorrect environment variable

## 12. How do you troubleshoot CrashLoopBackOff?
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs <pod-name> -c <container-name> --previous
kubectl get events --sort-by=.lastTimestamp
```

## 13. What is `ImagePullBackOff`?
Kubernetes cannot pull the container image and is retrying with increasing delay.

Causes:
- Wrong image name or tag
- Private registry authentication failure
- Network problem
- Registry unavailable
- Image does not exist

## 14. How do you troubleshoot ImagePullBackOff?
```bash
kubectl describe pod <pod-name>
kubectl get secret
kubectl get serviceaccount
```
Check image name, tag, `imagePullSecrets`, registry access, and node connectivity.

## 15. What does Pending mean?
A Pod is not yet running. It may be waiting for scheduling, storage binding, image preparation, or another prerequisite.

## 16. How do you troubleshoot Pending?
```bash
kubectl describe pod <pod-name>
kubectl get nodes
kubectl get pvc
kubectl get events --sort-by=.lastTimestamp
```

## 17. What is `ContainerCreating`?
The Pod has been assigned to a node, but containers are not ready yet. Possible causes include volume mounting, image pulling, networking, or runtime setup.

## 18. What is `Terminating` stuck Pod?
Possible causes:
- Finalizers
- Node unavailable
- Volume detach delay
- Kubelet failure
- Graceful termination blocked

Inspect:
```bash
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o yaml
```

## 19. How do you check application logs?
```bash
kubectl logs <pod-name>
kubectl logs -f <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs <pod-name> --previous
```

## 20. How do you inspect a Pod?
```bash
kubectl get pod <pod-name> -o wide
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o yaml
```

## 21. How do you execute commands inside a container?
```bash
kubectl exec -it <pod-name> -- sh
kubectl exec -it <pod-name> -c <container-name> -- sh
```

## 22. How do you test Service connectivity?
```bash
kubectl run debug --rm -it --image=curlimages/curl -- sh
curl http://service-name:port
```

## 23. How do you inspect Service endpoints?
```bash
kubectl get svc
kubectl get endpoints
kubectl get endpointslices
```

## 24. What causes a Service to have no endpoints?
- Selector does not match Pod labels.
- Pods are not Ready.
- Wrong Namespace.
- Port configuration mismatch.
- Pods are not running.

## 25. How do you troubleshoot DNS?
```bash
kubectl run dns-test --rm -it --image=busybox -- nslookup kubernetes.default
kubectl get pods -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
```

## 26. How do you inspect cluster events?
```bash
kubectl get events -A --sort-by=.lastTimestamp
kubectl get events -n <namespace>
```

## 27. What is a failed readiness probe?
The container is running but is not considered ready for traffic. Check endpoint path, port, application startup, dependencies, and timeout settings.

## 28. What are troubleshooting best practices?
1. Start with `kubectl get`.
2. Use `kubectl describe`.
3. Read current and previous logs.
4. Inspect Events.
5. Verify labels and selectors.
6. Check resources and scheduling.
7. Check Services and EndpointSlices.
8. Check DNS and network policies.
9. Validate configuration and Secrets.
10. Reproduce using a temporary debug Pod.

## 29. Explain the troubleshooting flow in an interview.
First identify the Pod status with `kubectl get pods`. Then inspect Events using `kubectl describe pod`, read current and previous logs, verify configuration, resources, probes, labels, Services, storage, DNS, and node health. Troubleshooting should proceed from the outer symptom to the underlying dependency rather than restarting blindly.
