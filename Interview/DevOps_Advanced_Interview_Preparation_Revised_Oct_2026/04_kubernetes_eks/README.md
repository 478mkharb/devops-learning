# Kubernetes & EKS — 50 Advanced Interview Questions

**Audience:** Mid-level/L2 DevOps Engineer (3–5 years).
**Reference date:** 18 October 2026.

**Focus:** Kubernetes & EKS production operations, troubleshooting, security, and reliability

## How to practise
- First answer the **interview-ready answer** aloud in 60–90 seconds.
- Then explain the technical reasoning, trade-offs, and evidence you would collect.
- Treat commands as diagnostic starting points; replace placeholders and validate permissions.
- Do not run destructive or mutating commands in production without an approved change plan.
- Use the follow-up to test whether you understand the failure mechanism rather than memorizing a script.

## Questions

### Q1. Thirty percent of new Pods remain Pending after a release, while existing replicas are healthy. How do you identify the scheduling constraint?

**Difficulty:** Intermediate

**Interview-ready answer:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity. Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Deployments, StatefulSets, DaemonSets, requests/limits, affinity, taints Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity.

**Production scenario:** Pods stay Pending. Check resource requests, node selectors, taints/tolerations, PVC binding, quotas, and topology constraints.

**Commands / checks:** ``kubectl get pods -A -o wide`; `kubectl describe pod`; `kubectl get events -A``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q2. A Pod is Running but the Service has no endpoints. What label, readiness, and EndpointSlice checks would you perform?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Deployments, StatefulSets, DaemonSets, requests/limits, affinity, taints Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** Pods stay Pending. Check resource requests, node selectors, taints/tolerations, PVC binding, quotas, and topology constraints.

**Commands / checks:** ``kubectl get pods -A -o wide`; `kubectl describe pod`; `kubectl get events -A``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q3. A container repeatedly restarts with OOMKilled. How do you distinguish an undersized memory limit from a leak or node pressure?

**Difficulty:** Advanced

**Interview-ready answer:** Check container and node events, requests/limits, cgroup metrics, node pressure, and recent workload changes. Raise limits only after distinguishing a leak or memory spike from under-provisioning. Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Deployments, StatefulSets, DaemonSets, requests/limits, affinity, taints Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Check container and node events, requests/limits, cgroup metrics, node pressure, and recent workload changes. Raise limits only after distinguishing a leak or memory spike from under-provisioning.

**Production scenario:** Pods stay Pending. Check resource requests, node selectors, taints/tolerations, PVC binding, quotas, and topology constraints.

**Commands / checks:** ``kubectl get pods -A -o wide`; `kubectl describe pod`; `kubectl get events -A``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q4. An HPA reports unknown CPU metrics. What components and resource requests must be working?

**Difficulty:** Intermediate

**Interview-ready answer:** Confirm Metrics Server or the custom metrics adapter, resource requests, HPA conditions, and whether the bottleneck is Pods or node capacity. Scaling replicas will not fix a saturated downstream dependency. Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Deployments, StatefulSets, DaemonSets, requests/limits, affinity, taints Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Confirm Metrics Server or the custom metrics adapter, resource requests, HPA conditions, and whether the bottleneck is Pods or node capacity. Scaling replicas will not fix a saturated downstream dependency.

**Production scenario:** Pods stay Pending. Check resource requests, node selectors, taints/tolerations, PVC binding, quotas, and topology constraints.

**Commands / checks:** ``kubectl get pods -A -o wide`; `kubectl describe pod`; `kubectl get events -A``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q5. A rollout stalls because the new Pods never become Ready. How do startup, readiness, and liveness probes differ in this diagnosis?

**Difficulty:** Advanced

**Interview-ready answer:** Use startup probes to allow initialization, readiness probes to control traffic eligibility, and liveness probes only for processes that need restart when stuck. Check probe path, port, thresholds, and startup timing. Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Deployments, StatefulSets, DaemonSets, requests/limits, affinity, taints Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Use startup probes to allow initialization, readiness probes to control traffic eligibility, and liveness probes only for processes that need restart when stuck. Check probe path, port, thresholds, and startup timing.

**Production scenario:** Pods stay Pending. Check resource requests, node selectors, taints/tolerations, PVC binding, quotas, and topology constraints.

**Commands / checks:** ``kubectl get pods -A -o wide`; `kubectl describe pod`; `kubectl get events -A``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q6. A PVC remains Pending after moving a StatefulSet to another zone. What storage topology and CSI checks matter?

**Difficulty:** Intermediate

**Interview-ready answer:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity. Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Deployments, StatefulSets, DaemonSets, requests/limits, affinity, taints Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity.

**Production scenario:** Pods stay Pending. Check resource requests, node selectors, taints/tolerations, PVC binding, quotas, and topology constraints.

**Commands / checks:** ``kubectl get pods -A -o wide`; `kubectl describe pod`; `kubectl get events -A``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q7. A NetworkPolicy is added and only DNS calls fail. How would you test policy selectors, DNS egress, and CNI support?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Deployments, StatefulSets, DaemonSets, requests/limits, affinity, taints Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** Pods stay Pending. Check resource requests, node selectors, taints/tolerations, PVC binding, quotas, and topology constraints.

**Commands / checks:** ``kubectl get pods -A -o wide`; `kubectl describe pod`; `kubectl get events -A``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q8. An EKS workload needs S3 access. How would you assign a workload identity and verify the Pod is not using node-role credentials?

**Difficulty:** Advanced

**Interview-ready answer:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage. Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Deployments, StatefulSets, DaemonSets, requests/limits, affinity, taints Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage.

**Production scenario:** Pods stay Pending. Check resource requests, node selectors, taints/tolerations, PVC binding, quotas, and topology constraints.

**Commands / checks:** ``kubectl get pods -A -o wide`; `kubectl describe pod`; `kubectl get events -A``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q9. Pods can reach each other by IP but service-name requests fail. How do you isolate CoreDNS, namespace search, and Service issues?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Deployments, StatefulSets, DaemonSets, requests/limits, affinity, taints Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** Pods stay Pending. Check resource requests, node selectors, taints/tolerations, PVC binding, quotas, and topology constraints.

**Commands / checks:** ``kubectl get pods -A -o wide`; `kubectl describe pod`; `kubectl get events -A``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q10. Design a safe Kubernetes node-group upgrade with disruption budgets, capacity checks, and workload validation.

**Difficulty:** Advanced

**Interview-ready answer:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage. Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Deployments, StatefulSets, DaemonSets, requests/limits, affinity, taints Choose workload controllers based on lifecycle semantics, set realistic requests, use limits carefully, and inspect scheduling events before changing cluster capacity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage.

**Production scenario:** Pods stay Pending. Check resource requests, node selectors, taints/tolerations, PVC binding, quotas, and topology constraints.

**Commands / checks:** ``kubectl get pods -A -o wide`; `kubectl describe pod`; `kubectl get events -A``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q11. A Deployment is healthy according to Kubernetes but its API returns 503 through the ingress. Which layers do you check and in what order?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths. A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** ClusterIP, NodePort, LoadBalancer, EndpointSlices, CoreDNS A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths.

**Production scenario:** Pod IP works but Service DNS fails. Check DNS resolution, namespace-qualified names, Service endpoints, dataplane, and NetworkPolicy.

**Commands / checks:** ``kubectl get svc,endpointslice`; `kubectl exec ... -- nslookup`; CoreDNS logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q12. A pod is stuck Terminating for 20 minutes. How do you investigate finalizers, node health, volume detach, and graceful shutdown?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect PVC events, StorageClass, access mode, PV zone topology, CSI controller/node plugin health, IAM permissions, and attachment state. EBS volumes are AZ-scoped. A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** ClusterIP, NodePort, LoadBalancer, EndpointSlices, CoreDNS A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect PVC events, StorageClass, access mode, PV zone topology, CSI controller/node plugin health, IAM permissions, and attachment state. EBS volumes are AZ-scoped.

**Production scenario:** Pod IP works but Service DNS fails. Check DNS resolution, namespace-qualified names, Service endpoints, dataplane, and NetworkPolicy.

**Commands / checks:** ``kubectl get svc,endpointslice`; `kubectl exec ... -- nslookup`; CoreDNS logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q13. A node becomes NotReady and workloads are evicted. How do you determine whether the cause is kubelet, networking, disk pressure, or cloud infrastructure?

**Difficulty:** Advanced

**Interview-ready answer:** Check container and node events, requests/limits, cgroup metrics, node pressure, and recent workload changes. Raise limits only after distinguishing a leak or memory spike from under-provisioning. A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** ClusterIP, NodePort, LoadBalancer, EndpointSlices, CoreDNS A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Check container and node events, requests/limits, cgroup metrics, node pressure, and recent workload changes. Raise limits only after distinguishing a leak or memory spike from under-provisioning.

**Production scenario:** Pod IP works but Service DNS fails. Check DNS resolution, namespace-qualified names, Service endpoints, dataplane, and NetworkPolicy.

**Commands / checks:** ``kubectl get svc,endpointslice`; `kubectl exec ... -- nslookup`; CoreDNS logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q14. A ClusterIP Service works from one namespace but not another. What DNS name and NetworkPolicy differences could explain it?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** ClusterIP, NodePort, LoadBalancer, EndpointSlices, CoreDNS A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** Pod IP works but Service DNS fails. Check DNS resolution, namespace-qualified names, Service endpoints, dataplane, and NetworkPolicy.

**Commands / checks:** ``kubectl get svc,endpointslice`; `kubectl exec ... -- nslookup`; CoreDNS logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q15. A container image pull fails only on private EKS nodes. How do you verify ECR permissions, endpoints, DNS, and node connectivity?

**Difficulty:** Advanced

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** ClusterIP, NodePort, LoadBalancer, EndpointSlices, CoreDNS A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** Pod IP works but Service DNS fails. Check DNS resolution, namespace-qualified names, Service endpoints, dataplane, and NetworkPolicy.

**Commands / checks:** ``kubectl get svc,endpointslice`; `kubectl exec ... -- nslookup`; CoreDNS logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q16. A PodDisruptionBudget prevents a node drain during a planned upgrade. How do you resolve it without causing avoidable downtime?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** ClusterIP, NodePort, LoadBalancer, EndpointSlices, CoreDNS A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** Pod IP works but Service DNS fails. Check DNS resolution, namespace-qualified names, Service endpoints, dataplane, and NetworkPolicy.

**Commands / checks:** ``kubectl get svc,endpointslice`; `kubectl exec ... -- nslookup`; CoreDNS logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q17. A Helm upgrade changes a resource field and fails with an immutable-field error. How do you plan the replacement safely?

**Difficulty:** Intermediate

**Interview-ready answer:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage. A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** ClusterIP, NodePort, LoadBalancer, EndpointSlices, CoreDNS A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage.

**Production scenario:** Pod IP works but Service DNS fails. Check DNS resolution, namespace-qualified names, Service endpoints, dataplane, and NetworkPolicy.

**Commands / checks:** ``kubectl get svc,endpointslice`; `kubectl exec ... -- nslookup`; CoreDNS logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q18. An admission policy blocks a legitimate deployment. How do you identify the violated rule and handle exceptions without disabling policy globally?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** ClusterIP, NodePort, LoadBalancer, EndpointSlices, CoreDNS A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Pod IP works but Service DNS fails. Check DNS resolution, namespace-qualified names, Service endpoints, dataplane, and NetworkPolicy.

**Commands / checks:** ``kubectl get svc,endpointslice`; `kubectl exec ... -- nslookup`; CoreDNS logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q19. A DaemonSet is missing from some nodes. What taints, node selectors, architecture, and scheduling events do you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity. A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** ClusterIP, NodePort, LoadBalancer, EndpointSlices, CoreDNS A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity.

**Production scenario:** Pod IP works but Service DNS fails. Check DNS resolution, namespace-qualified names, Service endpoints, dataplane, and NetworkPolicy.

**Commands / checks:** ``kubectl get svc,endpointslice`; `kubectl exec ... -- nslookup`; CoreDNS logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q20. How would you decide between Cluster Autoscaler and Karpenter for a workload with variable node requirements?

**Difficulty:** Advanced

**Interview-ready answer:** Confirm Metrics Server or the custom metrics adapter, resource requests, HPA conditions, and whether the bottleneck is Pods or node capacity. Scaling replicas will not fix a saturated downstream dependency. A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** ClusterIP, NodePort, LoadBalancer, EndpointSlices, CoreDNS A Service routes to matching eligible endpoints; DNS provides discovery. Validate selectors, readiness, ports, DNS, and CNI/policy behavior separately. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Confirm Metrics Server or the custom metrics adapter, resource requests, HPA conditions, and whether the bottleneck is Pods or node capacity. Scaling replicas will not fix a saturated downstream dependency.

**Production scenario:** Pod IP works but Service DNS fails. Check DNS resolution, namespace-qualified names, Service endpoints, dataplane, and NetworkPolicy.

**Commands / checks:** ``kubectl get svc,endpointslice`; `kubectl exec ... -- nslookup`; CoreDNS logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q21. A pod can resolve public DNS but cannot resolve a ClusterIP Service. What CoreDNS and cluster DNS checks would you run?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Readiness/liveness/startup probes, HPA, PDB, Cluster Autoscaler/Karpenter Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** HPA shows unknown metrics. Check Metrics Server or custom/external metrics adapters, API availability, requests, and RBAC.

**Commands / checks:** ``kubectl describe hpa`; `kubectl get pdb`; inspect metrics API and node autoscaler logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q22. An application has high latency only after scaling out. What connection pool, cold-start, cache, and downstream saturation effects would you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Readiness/liveness/startup probes, HPA, PDB, Cluster Autoscaler/Karpenter Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** HPA shows unknown metrics. Check Metrics Server or custom/external metrics adapters, API availability, requests, and RBAC.

**Commands / checks:** ``kubectl describe hpa`; `kubectl get pdb`; inspect metrics API and node autoscaler logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q23. A rolling update briefly drops all endpoints. What readiness, maxUnavailable, maxSurge, and capacity settings might be wrong?

**Difficulty:** Advanced

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Readiness/liveness/startup probes, HPA, PDB, Cluster Autoscaler/Karpenter Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** HPA shows unknown metrics. Check Metrics Server or custom/external metrics adapters, API availability, requests, and RBAC.

**Commands / checks:** ``kubectl describe hpa`; `kubectl get pdb`; inspect metrics API and node autoscaler logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q24. A Secret was committed to a Git repository and mounted in Kubernetes. What immediate response and long-term secret-management changes are required?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths. Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Readiness/liveness/startup probes, HPA, PDB, Cluster Autoscaler/Karpenter Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths.

**Production scenario:** HPA shows unknown metrics. Check Metrics Server or custom/external metrics adapters, API availability, requests, and RBAC.

**Commands / checks:** ``kubectl describe hpa`; `kubectl get pdb`; inspect metrics API and node autoscaler logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q25. A service account can list secrets across all namespaces. How do you trace and reduce the effective RBAC permissions?

**Difficulty:** Advanced

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Readiness/liveness/startup probes, HPA, PDB, Cluster Autoscaler/Karpenter Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** HPA shows unknown metrics. Check Metrics Server or custom/external metrics adapters, API availability, requests, and RBAC.

**Commands / checks:** ``kubectl describe hpa`; `kubectl get pdb`; inspect metrics API and node autoscaler logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q26. An EKS upgrade passes control-plane validation but CoreDNS fails afterward. What add-on compatibility and rollout evidence do you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Readiness/liveness/startup probes, HPA, PDB, Cluster Autoscaler/Karpenter Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** HPA shows unknown metrics. Check Metrics Server or custom/external metrics adapters, API availability, requests, and RBAC.

**Commands / checks:** ``kubectl describe hpa`; `kubectl get pdb`; inspect metrics API and node autoscaler logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q27. A node reports DiskPressure because container logs grew rapidly. How do you identify the source and prevent recurrence?

**Difficulty:** Intermediate

**Interview-ready answer:** Check container and node events, requests/limits, cgroup metrics, node pressure, and recent workload changes. Raise limits only after distinguishing a leak or memory spike from under-provisioning. Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Readiness/liveness/startup probes, HPA, PDB, Cluster Autoscaler/Karpenter Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Check container and node events, requests/limits, cgroup metrics, node pressure, and recent workload changes. Raise limits only after distinguishing a leak or memory spike from under-provisioning.

**Production scenario:** HPA shows unknown metrics. Check Metrics Server or custom/external metrics adapters, API availability, requests, and RBAC.

**Commands / checks:** ``kubectl describe hpa`; `kubectl get pdb`; inspect metrics API and node autoscaler logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q28. A StatefulSet Pod cannot attach its EBS volume after rescheduling. What access-mode, AZ, attachment, and CSI checks apply?

**Difficulty:** Advanced

**Interview-ready answer:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity. Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Readiness/liveness/startup probes, HPA, PDB, Cluster Autoscaler/Karpenter Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity.

**Production scenario:** HPA shows unknown metrics. Check Metrics Server or custom/external metrics adapters, API availability, requests, and RBAC.

**Commands / checks:** ``kubectl describe hpa`; `kubectl get pdb`; inspect metrics API and node autoscaler logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q29. A NetworkPolicy has no effect after deployment. What CNI implementation and policy enforcement assumptions must you verify?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths. Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Readiness/liveness/startup probes, HPA, PDB, Cluster Autoscaler/Karpenter Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths.

**Production scenario:** HPA shows unknown metrics. Check Metrics Server or custom/external metrics adapters, API availability, requests, and RBAC.

**Commands / checks:** ``kubectl describe hpa`; `kubectl get pdb`; inspect metrics API and node autoscaler logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q30. Explain how you would collect a minimal diagnostic bundle for a Kubernetes incident without exposing application secrets.

**Difficulty:** Advanced

**Interview-ready answer:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths. Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Readiness/liveness/startup probes, HPA, PDB, Cluster Autoscaler/Karpenter Use probes for their intended purpose: startup gates slow initialization, readiness controls traffic eligibility, and liveness detects a stuck process. Scale on relevant signals and protect disruptions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths.

**Production scenario:** HPA shows unknown metrics. Check Metrics Server or custom/external metrics adapters, API availability, requests, and RBAC.

**Commands / checks:** ``kubectl describe hpa`; `kubectl get pdb`; inspect metrics API and node autoscaler logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q31. An ingress returns 404 while direct Service access works. How do you inspect ingress class, host rules, path rewrites, and controller logs?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** RBAC, service accounts, Pod Security, secrets, network policies, EKS IAM Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** A CI runner can list secrets cluster-wide. Trace its identity and bindings, reduce permissions, rotate exposed secrets, and audit API activity.

**Commands / checks:** ``kubectl auth can-i`; `kubectl auth whoami` where supported; inspect RoleBindings and service account`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q32. Pods are evicted even though average node CPU is low. What memory, ephemeral-storage, and QoS signals do you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Check container and node events, requests/limits, cgroup metrics, node pressure, and recent workload changes. Raise limits only after distinguishing a leak or memory spike from under-provisioning. Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** RBAC, service accounts, Pod Security, secrets, network policies, EKS IAM Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Check container and node events, requests/limits, cgroup metrics, node pressure, and recent workload changes. Raise limits only after distinguishing a leak or memory spike from under-provisioning.

**Production scenario:** A CI runner can list secrets cluster-wide. Trace its identity and bindings, reduce permissions, rotate exposed secrets, and audit API activity.

**Commands / checks:** ``kubectl auth can-i`; `kubectl auth whoami` where supported; inspect RoleBindings and service account`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q33. A CronJob creates overlapping runs and corrupts data. How would you configure concurrency policy, deadlines, and idempotence?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** RBAC, service accounts, Pod Security, secrets, network policies, EKS IAM Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** A CI runner can list secrets cluster-wide. Trace its identity and bindings, reduce permissions, rotate exposed secrets, and audit API activity.

**Commands / checks:** ``kubectl auth can-i`; `kubectl auth whoami` where supported; inspect RoleBindings and service account`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q34. A liveness probe repeatedly restarts a slow-starting service. What probe design change would you make and why?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** RBAC, service accounts, Pod Security, secrets, network policies, EKS IAM Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** A CI runner can list secrets cluster-wide. Trace its identity and bindings, reduce permissions, rotate exposed secrets, and audit API activity.

**Commands / checks:** ``kubectl auth can-i`; `kubectl auth whoami` where supported; inspect RoleBindings and service account`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q35. An HPA scales out but latency continues to rise. How do you check whether the bottleneck is downstream rather than compute?

**Difficulty:** Advanced

**Interview-ready answer:** Confirm Metrics Server or the custom metrics adapter, resource requests, HPA conditions, and whether the bottleneck is Pods or node capacity. Scaling replicas will not fix a saturated downstream dependency. Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** RBAC, service accounts, Pod Security, secrets, network policies, EKS IAM Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Confirm Metrics Server or the custom metrics adapter, resource requests, HPA conditions, and whether the bottleneck is Pods or node capacity. Scaling replicas will not fix a saturated downstream dependency.

**Production scenario:** A CI runner can list secrets cluster-wide. Trace its identity and bindings, reduce permissions, rotate exposed secrets, and audit API activity.

**Commands / checks:** ``kubectl auth can-i`; `kubectl auth whoami` where supported; inspect RoleBindings and service account`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q36. A node IAM role has permissions that application Pods should not inherit. How do you separate node identity from workload identity?

**Difficulty:** Intermediate

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** RBAC, service accounts, Pod Security, secrets, network policies, EKS IAM Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** A CI runner can list secrets cluster-wide. Trace its identity and bindings, reduce permissions, rotate exposed secrets, and audit API activity.

**Commands / checks:** ``kubectl auth can-i`; `kubectl auth whoami` where supported; inspect RoleBindings and service account`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q37. A cluster upgrade removes an API version used by a Helm chart. How do you find deprecated APIs and validate manifests before upgrading?

**Difficulty:** Intermediate

**Interview-ready answer:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage. Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** RBAC, service accounts, Pod Security, secrets, network policies, EKS IAM Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage.

**Production scenario:** A CI runner can list secrets cluster-wide. Trace its identity and bindings, reduce permissions, rotate exposed secrets, and audit API activity.

**Commands / checks:** ``kubectl auth can-i`; `kubectl auth whoami` where supported; inspect RoleBindings and service account`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q38. A pod cannot access a mounted secret after a rotation. What update propagation and application reload behavior should you check?

**Difficulty:** Advanced

**Interview-ready answer:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths. Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** RBAC, service accounts, Pod Security, secrets, network policies, EKS IAM Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths.

**Production scenario:** A CI runner can list secrets cluster-wide. Trace its identity and bindings, reduce permissions, rotate exposed secrets, and audit API activity.

**Commands / checks:** ``kubectl auth can-i`; `kubectl auth whoami` where supported; inspect RoleBindings and service account`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q39. How would you implement namespace-level resource quotas without causing critical production workloads to be unschedulable?

**Difficulty:** Intermediate

**Interview-ready answer:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity. Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** RBAC, service accounts, Pod Security, secrets, network policies, EKS IAM Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity.

**Production scenario:** A CI runner can list secrets cluster-wide. Trace its identity and bindings, reduce permissions, rotate exposed secrets, and audit API activity.

**Commands / checks:** ``kubectl auth can-i`; `kubectl auth whoami` where supported; inspect RoleBindings and service account`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q40. A canary rollout shows increased 5xx errors but Pods remain Ready. What metrics and rollout controls would you use to halt promotion?

**Difficulty:** Advanced

**Interview-ready answer:** Confirm Metrics Server or the custom metrics adapter, resource requests, HPA conditions, and whether the bottleneck is Pods or node capacity. Scaling replicas will not fix a saturated downstream dependency. Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** RBAC, service accounts, Pod Security, secrets, network policies, EKS IAM Apply least privilege to Kubernetes identities and AWS identities, use namespace boundaries and admission controls, and avoid embedding secrets in images or Git. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Confirm Metrics Server or the custom metrics adapter, resource requests, HPA conditions, and whether the bottleneck is Pods or node capacity. Scaling replicas will not fix a saturated downstream dependency.

**Production scenario:** A CI runner can list secrets cluster-wide. Trace its identity and bindings, reduce permissions, rotate exposed secrets, and audit API activity.

**Commands / checks:** ``kubectl auth can-i`; `kubectl auth whoami` where supported; inspect RoleBindings and service account`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q41. An EKS cluster has intermittent DNS timeouts during peak traffic. How would you inspect CoreDNS capacity, throttling, cache behavior, and node networking?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** PVC/PV/StorageClass, CSI drivers, version skew, add-ons, rollout recovery Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** PVC is Pending after moving workloads. Check StorageClass, access mode, zone topology, CSI controller/node plugin, quota, and IAM permissions.

**Commands / checks:** ``kubectl get pvc,pv,sc`; `kubectl get nodes`; EKS cluster insights and add-on status`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q42. A team runs privileged containers to fix a filesystem permission issue. What safer alternatives would you explore?

**Difficulty:** Intermediate

**Interview-ready answer:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths. Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** PVC/PV/StorageClass, CSI drivers, version skew, add-ons, rollout recovery Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Verify the CNI enforces the relevant NetworkPolicy API, selectors match intended Pods/namespaces, and the effective RBAC/IAM permissions are least-privilege. Validate both allowed and denied paths.

**Production scenario:** PVC is Pending after moving workloads. Check StorageClass, access mode, zone topology, CSI controller/node plugin, quota, and IAM permissions.

**Commands / checks:** ``kubectl get pvc,pv,sc`; `kubectl get nodes`; EKS cluster insights and add-on status`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q43. A PVC deletion unexpectedly removes data. Which reclaim policy, StatefulSet, and storage-backup settings should have been reviewed?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect PVC events, StorageClass, access mode, PV zone topology, CSI controller/node plugin health, IAM permissions, and attachment state. EBS volumes are AZ-scoped. Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** PVC/PV/StorageClass, CSI drivers, version skew, add-ons, rollout recovery Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect PVC events, StorageClass, access mode, PV zone topology, CSI controller/node plugin health, IAM permissions, and attachment state. EBS volumes are AZ-scoped.

**Production scenario:** PVC is Pending after moving workloads. Check StorageClass, access mode, zone topology, CSI controller/node plugin, quota, and IAM permissions.

**Commands / checks:** ``kubectl get pvc,pv,sc`; `kubectl get nodes`; EKS cluster insights and add-on status`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q44. An EKS managed node group upgrade stalls. What launch template, capacity, PDB, subnet IP, and bootstrap checks do you perform?

**Difficulty:** Intermediate

**Interview-ready answer:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage. Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** PVC/PV/StorageClass, CSI drivers, version skew, add-ons, rollout recovery Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage.

**Production scenario:** PVC is Pending after moving workloads. Check StorageClass, access mode, zone topology, CSI controller/node plugin, quota, and IAM permissions.

**Commands / checks:** ``kubectl get pvc,pv,sc`; `kubectl get nodes`; EKS cluster insights and add-on status`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q45. A service is reachable inside the cluster but not through the public ALB. How do you isolate target type, health checks, security groups, and ingress rules?

**Difficulty:** Advanced

**Interview-ready answer:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS. Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** PVC/PV/StorageClass, CSI drivers, version skew, add-ons, rollout recovery Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Verify Service selectors and EndpointSlices, then readiness and port/targetPort. For name failures, inspect CoreDNS, namespace-qualified names, search paths, and NetworkPolicy egress to DNS.

**Production scenario:** PVC is Pending after moving workloads. Check StorageClass, access mode, zone topology, CSI controller/node plugin, quota, and IAM permissions.

**Commands / checks:** ``kubectl get pvc,pv,sc`; `kubectl get nodes`; EKS cluster insights and add-on status`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q46. How would you define a minimum set of Kubernetes alerts that catch user-impacting failure without paging on every restart?

**Difficulty:** Intermediate

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** PVC/PV/StorageClass, CSI drivers, version skew, add-ons, rollout recovery Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** PVC is Pending after moving workloads. Check StorageClass, access mode, zone topology, CSI controller/node plugin, quota, and IAM permissions.

**Commands / checks:** ``kubectl get pvc,pv,sc`; `kubectl get nodes`; EKS cluster insights and add-on status`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q47. A rollout uses a mutable image tag and nodes run different code. How do you enforce immutable artifact identity?

**Difficulty:** Intermediate

**Interview-ready answer:** For image pulls, check registry reachability and node execution role; for runtime AWS API calls, check workload identity and the Pod's actual credential source. These are separate identity paths. Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** PVC/PV/StorageClass, CSI drivers, version skew, add-ons, rollout recovery Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** For image pulls, check registry reachability and node execution role; for runtime AWS API calls, check workload identity and the Pod's actual credential source. These are separate identity paths.

**Production scenario:** PVC is Pending after moving workloads. Check StorageClass, access mode, zone topology, CSI controller/node plugin, quota, and IAM permissions.

**Commands / checks:** ``kubectl get pvc,pv,sc`; `kubectl get nodes`; EKS cluster insights and add-on status`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q48. A pod is stuck Pending due to insufficient IP addresses in a VPC subnet. What EKS CNI and subnet-capacity options do you evaluate?

**Difficulty:** Advanced

**Interview-ready answer:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity. Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** PVC/PV/StorageClass, CSI drivers, version skew, add-ons, rollout recovery Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Read Pod events first. Distinguish insufficient CPU/memory, taints, affinity, quota, PVC binding, topology constraints, and VPC CNI IP exhaustion before changing capacity.

**Production scenario:** PVC is Pending after moving workloads. Check StorageClass, access mode, zone topology, CSI controller/node plugin, quota, and IAM permissions.

**Commands / checks:** ``kubectl get pvc,pv,sc`; `kubectl get nodes`; EKS cluster insights and add-on status`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q49. An application needs to write to S3 and publish to SQS. How do you keep permissions narrow and audit its actions?

**Difficulty:** Intermediate

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** PVC/PV/StorageClass, CSI drivers, version skew, add-ons, rollout recovery Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** PVC is Pending after moving workloads. Check StorageClass, access mode, zone topology, CSI controller/node plugin, quota, and IAM permissions.

**Commands / checks:** ``kubectl get pvc,pv,sc`; `kubectl get nodes`; EKS cluster insights and add-on status`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q50. Walk through your response when a cluster upgrade causes a production outage and a simple control-plane downgrade is not available.

**Difficulty:** Advanced

**Interview-ready answer:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage. Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** PVC/PV/StorageClass, CSI drivers, version skew, add-ons, rollout recovery Validate storage topology and CSI health, and upgrade clusters using supported version-skew and add-on procedures. Test APIs, workloads, capacity, and recovery before production upgrades. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Check EKS version support, deprecated APIs, add-on compatibility, node capacity, PDBs, and version skew. Upgrade in supported order and validate workload behavior after each stage.

**Production scenario:** PVC is Pending after moving workloads. Check StorageClass, access mode, zone topology, CSI controller/node plugin, quota, and IAM permissions.

**Commands / checks:** ``kubectl get pvc,pv,sc`; `kubectl get nodes`; EKS cluster insights and add-on status`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

## Official references

- [Kubernetes releases](https://kubernetes.io/releases/)
- [EKS version lifecycle](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html)
- [EKS Kubernetes 1.37 announcement](https://aws.amazon.com/about-aws/whats-new/2026/10/amazon-eks-distro-kubernetes-version-1-37/)

## Version and safety note

This guide is dated 18 October 2026. Release status, feature maturity, cloud-region availability, plugins, and supported versions change. Check the linked official documentation against the exact versions and architecture you use before applying any production recommendation.
