# Kubernetes Autoscaling Interview Questions and Answers

## 1. What is Kubernetes autoscaling?
Autoscaling automatically adjusts workload replicas, Pod resources, or cluster nodes according to demand and constraints.

Main mechanisms:
- Horizontal Pod Autoscaler
- Vertical Pod Autoscaler
- Cluster Autoscaler
- Event-driven autoscaling tools

## 2. What is HPA?
Horizontal Pod Autoscaler changes the number of Pod replicas in a workload such as a Deployment or StatefulSet.

## 3. What metrics can HPA use?
Depending on configuration:
- CPU utilization
- Memory utilization
- Custom metrics
- External metrics

## 4. Example HPA.
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

## 5. What is required for CPU-based HPA?
Usually:
- Metrics Server
- CPU requests on containers
- A supported workload target
- Correct HPA configuration

CPU utilization is calculated relative to requested CPU.

## 6. What is VPA?
Vertical Pod Autoscaler adjusts Pod resource requests and limits based on observed usage and recommendations.

It can:
- Recommend resources
- Update resources
- Automatically apply resource changes depending on mode

## 7. What are VPA modes?
Common modes:
- `Off`
- `Initial`
- `Recreate`
- `Auto` in supported implementations/versions

## 8. What is Cluster Autoscaler?
Cluster Autoscaler adds nodes when Pods cannot be scheduled due to insufficient capacity and removes underutilized nodes when safe.

## 9. Difference between HPA, VPA and Cluster Autoscaler?
| Tool | Scales |
|---|---|
| HPA | Number of Pods |
| VPA | Pod resource requests/limits |
| Cluster Autoscaler | Number of nodes |

## 10. What is the relationship between HPA and Cluster Autoscaler?
HPA creates more Pods. If existing nodes cannot fit them, Cluster Autoscaler may add nodes. The Scheduler then places the Pods.

## 11. What is KEDA?
KEDA is an event-driven autoscaling component that can scale workloads based on external event sources such as queues, streams, and metrics.

## 12. What is scale-to-zero?
Scale-to-zero means reducing a workload to zero replicas when there is no demand. Native HPA behavior and event-driven tools have different support and constraints; KEDA is commonly used for event-driven scale-to-zero.

## 13. What is stabilization in HPA?
Stabilization prevents rapid oscillation by delaying scale-up or scale-down decisions based on recent recommendations.

## 14. What is HPA behavior?
HPA `behavior` configures scaling policies, stabilization windows, and rate limits.

## 15. What causes HPA to show unknown metrics?
- Metrics Server unavailable
- Missing resource requests
- Metrics API failure
- Incorrect target reference
- Permission or API aggregation issue

## 16. How do you troubleshoot HPA?
```bash
kubectl get hpa
kubectl describe hpa <name>
kubectl top pods
kubectl get apiservice
kubectl get deployment metrics-server -n kube-system
```

## 17. Why does HPA not scale on CPU?
Check CPU requests, Metrics Server, actual load, target type, min/max replicas, and whether the workload is already at its maximum.

## 18. What is a resource request’s role in HPA?
For utilization targets, HPA compares observed usage with the requested resource amount. Incorrect requests can produce misleading scaling behavior.

## 19. What is overprovisioning?
Overprovisioning reserves more resources than the application needs, reducing cluster utilization and increasing cost.

## 20. What is underprovisioning?
Underprovisioning gives too few resources, causing throttling, OOMKills, latency, evictions, or unstable scaling.

## 21. What are autoscaling best practices?
- Set realistic requests.
- Use meaningful min/max replicas.
- Monitor scaling latency.
- Configure stabilization.
- Avoid conflicting HPA and VPA policies.
- Test metrics.
- Use PodDisruptionBudgets.
- Ensure cluster capacity.
- Monitor cost.
- Test failure and burst scenarios.

## 22. Explain autoscaling in an interview.
HPA changes the number of Pods, VPA adjusts Pod resources, and Cluster Autoscaler changes the number of nodes. HPA commonly uses CPU, memory, custom, or external metrics. When HPA creates Pods that cannot fit, Cluster Autoscaler can add nodes, after which the Scheduler places the Pods. Autoscaling requires accurate metrics, realistic resource requests, suitable limits, and careful stabilization to avoid oscillation.
