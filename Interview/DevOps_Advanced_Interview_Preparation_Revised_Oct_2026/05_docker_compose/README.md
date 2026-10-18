# Docker & Docker Compose — 50 Advanced Interview Questions

**Audience:** Mid-level/L2 DevOps Engineer (3–5 years).
**Reference date:** 18 October 2026.

**Focus:** Docker & Docker Compose production operations, troubleshooting, security, and reliability

## How to practise
- First answer the **interview-ready answer** aloud in 60–90 seconds.
- Then explain the technical reasoning, trade-offs, and evidence you would collect.
- Treat commands as diagnostic starting points; replace placeholders and validate permissions.
- Do not run destructive or mutating commands in production without an approved change plan.
- Use the follow-up to test whether you understand the failure mechanism rather than memorizing a script.

## Questions

### Q1. A container exits with code 137 after a traffic spike. How do you distinguish an OOM kill from a manual kill or host pressure?

**Difficulty:** Intermediate

**Interview-ready answer:** Exit code 137 commonly indicates SIGKILL and may be an OOM kill, but confirm with Docker state, host kernel logs, and memory metrics before concluding. Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Dockerfile layers, multi-stage builds, cache, pinning, image digest Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Exit code 137 commonly indicates SIGKILL and may be an OOM kill, but confirm with Docker state, host kernel logs, and memory metrics before concluding.

**Production scenario:** An image is unexpectedly huge. Inspect layers and build context, exclude secrets/artifacts, and move compilers/build dependencies to a build stage.

**Commands / checks:** ``docker build`; `docker history`; `docker image inspect`; image scanning`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q2. A Compose service cannot reach another service using localhost. Explain the correct service discovery and port model.

**Difficulty:** Intermediate

**Interview-ready answer:** Inside a container, localhost refers to that same container. Use the Compose service name and the container's listening port, and verify both services share the intended network. Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Dockerfile layers, multi-stage builds, cache, pinning, image digest Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inside a container, localhost refers to that same container. Use the Compose service name and the container's listening port, and verify both services share the intended network.

**Production scenario:** An image is unexpectedly huge. Inspect layers and build context, exclude secrets/artifacts, and move compilers/build dependencies to a build stage.

**Commands / checks:** ``docker build`; `docker history`; `docker image inspect`; image scanning`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q3. An image is 2 GB because build tools and caches remain in the final image. How would you reduce it safely?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Dockerfile layers, multi-stage builds, cache, pinning, image digest Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** An image is unexpectedly huge. Inspect layers and build context, exclude secrets/artifacts, and move compilers/build dependencies to a build stage.

**Commands / checks:** ``docker build`; `docker history`; `docker image inspect`; image scanning`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q4. A container's data disappears after recreation. How do you verify whether data was stored in the writable layer or a named volume?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Dockerfile layers, multi-stage builds, cache, pinning, image digest Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** An image is unexpectedly huge. Inspect layers and build context, exclude secrets/artifacts, and move compilers/build dependencies to a build stage.

**Commands / checks:** ``docker build`; `docker history`; `docker image inspect`; image scanning`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q5. A container works interactively but fails under Compose. Which entrypoint, working directory, environment, user, and signal differences do you inspect?

**Difficulty:** Advanced

**Interview-ready answer:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior. Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Dockerfile layers, multi-stage builds, cache, pinning, image digest Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior.

**Production scenario:** An image is unexpectedly huge. Inspect layers and build context, exclude secrets/artifacts, and move compilers/build dependencies to a build stage.

**Commands / checks:** ``docker build`; `docker history`; `docker image inspect`; image scanning`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q6. A database container starts before its schema is ready and the API crashes. How do you coordinate readiness and application retries?

**Difficulty:** Intermediate

**Interview-ready answer:** Confirm the application's data path is mounted to a named volume or intended bind mount, verify ownership and mount options, and test restore rather than assuming persistence. Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Dockerfile layers, multi-stage builds, cache, pinning, image digest Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Confirm the application's data path is mounted to a named volume or intended bind mount, verify ownership and mount options, and test restore rather than assuming persistence.

**Production scenario:** An image is unexpectedly huge. Inspect layers and build context, exclude secrets/artifacts, and move compilers/build dependencies to a build stage.

**Commands / checks:** ``docker build`; `docker history`; `docker image inspect`; image scanning`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q7. A Docker build accidentally includes `.env` and Git credentials in an image layer. What immediate and preventive actions are needed?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Dockerfile layers, multi-stage builds, cache, pinning, image digest Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** An image is unexpectedly huge. Inspect layers and build context, exclude secrets/artifacts, and move compilers/build dependencies to a build stage.

**Commands / checks:** ``docker build`; `docker history`; `docker image inspect`; image scanning`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q8. Only one Compose service cannot resolve another service name. How do you inspect network membership, aliases, and DNS?

**Difficulty:** Advanced

**Interview-ready answer:** Inside a container, localhost refers to that same container. Use the Compose service name and the container's listening port, and verify both services share the intended network. Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Dockerfile layers, multi-stage builds, cache, pinning, image digest Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inside a container, localhost refers to that same container. Use the Compose service name and the container's listening port, and verify both services share the intended network.

**Production scenario:** An image is unexpectedly huge. Inspect layers and build context, exclude secrets/artifacts, and move compilers/build dependencies to a build stage.

**Commands / checks:** ``docker build`; `docker history`; `docker image inspect`; image scanning`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q9. A container is healthy but the application is unreachable from the host. How do published ports, bind address, and firewall rules interact?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare the process's bind address, container port, published host port, host firewall, and network mode. A port mapping cannot make an app listening only on the wrong interface reachable. Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Dockerfile layers, multi-stage builds, cache, pinning, image digest Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Compare the process's bind address, container port, published host port, host firewall, and network mode. A port mapping cannot make an app listening only on the wrong interface reachable.

**Production scenario:** An image is unexpectedly huge. Inspect layers and build context, exclude secrets/artifacts, and move compilers/build dependencies to a build stage.

**Commands / checks:** ``docker build`; `docker history`; `docker image inspect`; image scanning`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q10. How would you make a Compose-based developer environment reproducible across Linux laptops and CI?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Dockerfile layers, multi-stage builds, cache, pinning, image digest Use small trusted base images, multi-stage builds, deterministic dependencies, non-root users, and immutable digests for production promotion. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** An image is unexpectedly huge. Inspect layers and build context, exclude secrets/artifacts, and move compilers/build dependencies to a build stage.

**Commands / checks:** ``docker build`; `docker history`; `docker image inspect`; image scanning`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q11. A multi-stage build still contains a compiler and package cache. How would you inspect and fix the Dockerfile?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Entrypoints, signals, health checks, CPU/memory limits, logs Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** Container exits with code 137. Investigate OOM kill, host memory pressure, container limits, and whether the process handles SIGTERM correctly.

**Commands / checks:** ``docker inspect`; `docker stats`; `docker logs --tail 200`; `docker events``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q12. A service is killed during `docker compose down`. How should the application handle SIGTERM and graceful shutdown?

**Difficulty:** Intermediate

**Interview-ready answer:** Exit code 137 commonly indicates SIGKILL and may be an OOM kill, but confirm with Docker state, host kernel logs, and memory metrics before concluding. Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Entrypoints, signals, health checks, CPU/memory limits, logs Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Exit code 137 commonly indicates SIGKILL and may be an OOM kill, but confirm with Docker state, host kernel logs, and memory metrics before concluding.

**Production scenario:** Container exits with code 137. Investigate OOM kill, host memory pressure, container limits, and whether the process handles SIGTERM correctly.

**Commands / checks:** ``docker inspect`; `docker stats`; `docker logs --tail 200`; `docker events``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q13. A bind mount works on one Linux host but permissions fail on another. What UID/GID and host filesystem differences matter?

**Difficulty:** Advanced

**Interview-ready answer:** Confirm the application's data path is mounted to a named volume or intended bind mount, verify ownership and mount options, and test restore rather than assuming persistence. Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Entrypoints, signals, health checks, CPU/memory limits, logs Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Confirm the application's data path is mounted to a named volume or intended bind mount, verify ownership and mount options, and test restore rather than assuming persistence.

**Production scenario:** Container exits with code 137. Investigate OOM kill, host memory pressure, container limits, and whether the process handles SIGTERM correctly.

**Commands / checks:** ``docker inspect`; `docker stats`; `docker logs --tail 200`; `docker events``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q14. A Compose deployment leaks database credentials through `docker inspect`. How would you change secret delivery and environment configuration?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Entrypoints, signals, health checks, CPU/memory limits, logs Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** Container exits with code 137. Investigate OOM kill, host memory pressure, container limits, and whether the process handles SIGTERM correctly.

**Commands / checks:** ``docker inspect`; `docker stats`; `docker logs --tail 200`; `docker events``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q15. A container can reach the internet but cannot reach a service on the host. What host-gateway and network-mode considerations apply?

**Difficulty:** Advanced

**Interview-ready answer:** Inside a container, localhost refers to that same container. Use the Compose service name and the container's listening port, and verify both services share the intended network. Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Entrypoints, signals, health checks, CPU/memory limits, logs Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inside a container, localhost refers to that same container. Use the Compose service name and the container's listening port, and verify both services share the intended network.

**Production scenario:** Container exits with code 137. Investigate OOM kill, host memory pressure, container limits, and whether the process handles SIGTERM correctly.

**Commands / checks:** ``docker inspect`; `docker stats`; `docker logs --tail 200`; `docker events``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q16. A Compose health check passes even though the API cannot serve real requests. How would you design a meaningful health check?

**Difficulty:** Intermediate

**Interview-ready answer:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior. Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Entrypoints, signals, health checks, CPU/memory limits, logs Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior.

**Production scenario:** Container exits with code 137. Investigate OOM kill, host memory pressure, container limits, and whether the process handles SIGTERM correctly.

**Commands / checks:** ``docker inspect`; `docker stats`; `docker logs --tail 200`; `docker events``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q17. A developer uses `docker system prune -a` on a shared build host. What data could be removed and how should cleanup be governed?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Entrypoints, signals, health checks, CPU/memory limits, logs Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** Container exits with code 137. Investigate OOM kill, host memory pressure, container limits, and whether the process handles SIGTERM correctly.

**Commands / checks:** ``docker inspect`; `docker stats`; `docker logs --tail 200`; `docker events``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q18. A container image pulls successfully on one machine but fails on another architecture. How do you inspect manifest lists and platform selection?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Entrypoints, signals, health checks, CPU/memory limits, logs Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** Container exits with code 137. Investigate OOM kill, host memory pressure, container limits, and whether the process handles SIGTERM correctly.

**Commands / checks:** ``docker inspect`; `docker stats`; `docker logs --tail 200`; `docker events``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q19. A Dockerized process ignores shutdown and takes too long to stop. How do ENTRYPOINT, shell form, exec form, and init behavior affect it?

**Difficulty:** Intermediate

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Entrypoints, signals, health checks, CPU/memory limits, logs Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Container exits with code 137. Investigate OOM kill, host memory pressure, container limits, and whether the process handles SIGTERM correctly.

**Commands / checks:** ``docker inspect`; `docker stats`; `docker logs --tail 200`; `docker events``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q20. A Compose service loops restarting because its dependency is unavailable. How do you prevent retry storms and preserve useful diagnostics?

**Difficulty:** Advanced

**Interview-ready answer:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior. Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Entrypoints, signals, health checks, CPU/memory limits, logs Run one primary responsibility per container, handle termination signals, define health checks, and configure resource limits based on measured behavior. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior.

**Production scenario:** Container exits with code 137. Investigate OOM kill, host memory pressure, container limits, and whether the process handles SIGTERM correctly.

**Commands / checks:** ``docker inspect`; `docker stats`; `docker logs --tail 200`; `docker events``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q21. Your image scan reports a vulnerable base image. How do you patch while preserving reproducibility and avoiding unnecessary layer growth?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bridge networks, published ports, DNS, service names, host binding Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** One container cannot reach another using localhost. `localhost` refers to the same container; use the Compose service name and correct container port.

**Commands / checks:** ``docker network inspect`; `docker compose exec`; `getent hosts <service>``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q22. A container needs only read access to a config file but runs as root. How would you reduce privileges and set filesystem permissions?

**Difficulty:** Intermediate

**Interview-ready answer:** Run as a non-root UID where possible, grant only required capabilities, use read-only root filesystems where compatible, and set explicit volume ownership and writable paths. Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bridge networks, published ports, DNS, service names, host binding Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Run as a non-root UID where possible, grant only required capabilities, use read-only root filesystems where compatible, and set explicit volume ownership and writable paths.

**Production scenario:** One container cannot reach another using localhost. `localhost` refers to the same container; use the Compose service name and correct container port.

**Commands / checks:** ``docker network inspect`; `docker compose exec`; `getent hosts <service>``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q23. An application writes logs to a file inside the container and the host log system sees nothing. What logging design would you use?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bridge networks, published ports, DNS, service names, host binding Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** One container cannot reach another using localhost. `localhost` refers to the same container; use the Compose service name and correct container port.

**Commands / checks:** ``docker network inspect`; `docker compose exec`; `getent hosts <service>``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q24. A named volume contains stale database data after a test reset. How do you reset safely without deleting production data?

**Difficulty:** Intermediate

**Interview-ready answer:** Confirm the application's data path is mounted to a named volume or intended bind mount, verify ownership and mount options, and test restore rather than assuming persistence. Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bridge networks, published ports, DNS, service names, host binding Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Confirm the application's data path is mounted to a named volume or intended bind mount, verify ownership and mount options, and test restore rather than assuming persistence.

**Production scenario:** One container cannot reach another using localhost. `localhost` refers to the same container; use the Compose service name and correct container port.

**Commands / checks:** ``docker network inspect`; `docker compose exec`; `getent hosts <service>``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q25. Two Compose projects use the same default network name and interfere with each other. How would you isolate project networks?

**Difficulty:** Advanced

**Interview-ready answer:** Inside a container, localhost refers to that same container. Use the Compose service name and the container's listening port, and verify both services share the intended network. Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bridge networks, published ports, DNS, service names, host binding Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inside a container, localhost refers to that same container. Use the Compose service name and the container's listening port, and verify both services share the intended network.

**Production scenario:** One container cannot reach another using localhost. `localhost` refers to the same container; use the Compose service name and correct container port.

**Commands / checks:** ``docker network inspect`; `docker compose exec`; `getent hosts <service>``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q26. A build cache appears to contain output from an untrusted branch. How do you separate caches by trust boundary?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bridge networks, published ports, DNS, service names, host binding Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** One container cannot reach another using localhost. `localhost` refers to the same container; use the Compose service name and correct container port.

**Commands / checks:** ``docker network inspect`; `docker compose exec`; `getent hosts <service>``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q27. A container cannot bind to its configured port even though the port is published. What process and container-port checks do you run?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare the process's bind address, container port, published host port, host firewall, and network mode. A port mapping cannot make an app listening only on the wrong interface reachable. Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bridge networks, published ports, DNS, service names, host binding Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Compare the process's bind address, container port, published host port, host firewall, and network mode. A port mapping cannot make an app listening only on the wrong interface reachable.

**Production scenario:** One container cannot reach another using localhost. `localhost` refers to the same container; use the Compose service name and correct container port.

**Commands / checks:** ``docker network inspect`; `docker compose exec`; `getent hosts <service>``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q28. A developer copies a private key into an image and deletes it in a later layer. Why is the key still exposed and how do you remediate?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bridge networks, published ports, DNS, service names, host binding Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** One container cannot reach another using localhost. `localhost` refers to the same container; use the Compose service name and correct container port.

**Commands / checks:** ``docker network inspect`; `docker compose exec`; `getent hosts <service>``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q29. How would you enforce read-only root filesystems and writable temporary paths without breaking the application?

**Difficulty:** Intermediate

**Interview-ready answer:** Run as a non-root UID where possible, grant only required capabilities, use read-only root filesystems where compatible, and set explicit volume ownership and writable paths. Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bridge networks, published ports, DNS, service names, host binding Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Run as a non-root UID where possible, grant only required capabilities, use read-only root filesystems where compatible, and set explicit volume ownership and writable paths.

**Production scenario:** One container cannot reach another using localhost. `localhost` refers to the same container; use the Compose service name and correct container port.

**Commands / checks:** ``docker network inspect`; `docker compose exec`; `getent hosts <service>``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q30. How do you decide which containers need CPU/memory limits in a Compose development stack?

**Difficulty:** Advanced

**Interview-ready answer:** Exit code 137 commonly indicates SIGKILL and may be an OOM kill, but confirm with Docker state, host kernel logs, and memory metrics before concluding. Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Bridge networks, published ports, DNS, service names, host binding Containers on the same Compose network should communicate using service DNS names and container ports. Publish ports only when host access is needed. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Exit code 137 commonly indicates SIGKILL and may be an OOM kill, but confirm with Docker state, host kernel logs, and memory metrics before concluding.

**Production scenario:** One container cannot reach another using localhost. `localhost` refers to the same container; use the Compose service name and correct container port.

**Commands / checks:** ``docker network inspect`; `docker compose exec`; `getent hosts <service>``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q31. A Compose database service has no health check, and application startup is flaky. What health semantics and dependency conditions should be used?

**Difficulty:** Intermediate

**Interview-ready answer:** Confirm the application's data path is mounted to a named volume or intended bind mount, verify ownership and mount options, and test restore rather than assuming persistence. Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Volumes, bind mounts, secrets, env files, backups Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Confirm the application's data path is mounted to a named volume or intended bind mount, verify ownership and mount options, and test restore rather than assuming persistence.

**Production scenario:** Data disappears after recreation. Check whether data was written to the container writable layer instead of a persistent volume.

**Commands / checks:** ``docker volume inspect`; `docker compose config`; inspect mount declarations`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q32. A local stack starts but fails after reboot because the database volume ownership changed. How do you make ownership and initialization predictable?

**Difficulty:** Intermediate

**Interview-ready answer:** Confirm the application's data path is mounted to a named volume or intended bind mount, verify ownership and mount options, and test restore rather than assuming persistence. Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Volumes, bind mounts, secrets, env files, backups Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Confirm the application's data path is mounted to a named volume or intended bind mount, verify ownership and mount options, and test restore rather than assuming persistence.

**Production scenario:** Data disappears after recreation. Check whether data was written to the container writable layer instead of a persistent volume.

**Commands / checks:** ``docker volume inspect`; `docker compose config`; inspect mount declarations`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q33. A container has a DNS resolution delay only on one network. What resolver, embedded DNS, and host configuration checks are useful?

**Difficulty:** Advanced

**Interview-ready answer:** Inside a container, localhost refers to that same container. Use the Compose service name and the container's listening port, and verify both services share the intended network. Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Volumes, bind mounts, secrets, env files, backups Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inside a container, localhost refers to that same container. Use the Compose service name and the container's listening port, and verify both services share the intended network.

**Production scenario:** Data disappears after recreation. Check whether data was written to the container writable layer instead of a persistent volume.

**Commands / checks:** ``docker volume inspect`; `docker compose config`; inspect mount declarations`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q34. A Compose file includes hard-coded private IPs. How would you replace them with stable service discovery and configuration?

**Difficulty:** Intermediate

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Volumes, bind mounts, secrets, env files, backups Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Data disappears after recreation. Check whether data was written to the container writable layer instead of a persistent volume.

**Commands / checks:** ``docker volume inspect`; `docker compose config`; inspect mount declarations`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q35. A service publishes `0.0.0.0:5432` on a laptop. What is the exposure risk and how would you limit host binding?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Volumes, bind mounts, secrets, env files, backups Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Data disappears after recreation. Check whether data was written to the container writable layer instead of a persistent volume.

**Commands / checks:** ``docker volume inspect`; `docker compose config`; inspect mount declarations`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q36. How would you handle schema migrations in a Compose environment so multiple app replicas do not race?

**Difficulty:** Intermediate

**Interview-ready answer:** Treat schema migration and stateful upgrades as a separate controlled step. Verify backups, compatibility, and rollback/forward-fix paths before replacing containers. Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Volumes, bind mounts, secrets, env files, backups Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Treat schema migration and stateful upgrades as a separate controlled step. Verify backups, compatibility, and rollback/forward-fix paths before replacing containers.

**Production scenario:** Data disappears after recreation. Check whether data was written to the container writable layer instead of a persistent volume.

**Commands / checks:** ``docker volume inspect`; `docker compose config`; inspect mount declarations`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q37. A health check is too aggressive and causes restart loops during slow startup. How do interval, timeout, retries, and start period interact?

**Difficulty:** Intermediate

**Interview-ready answer:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior. Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Volumes, bind mounts, secrets, env files, backups Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior.

**Production scenario:** Data disappears after recreation. Check whether data was written to the container writable layer instead of a persistent volume.

**Commands / checks:** ``docker volume inspect`; `docker compose config`; inspect mount declarations`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q38. How do you distinguish an application failure from a Docker daemon or host kernel issue?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Volumes, bind mounts, secrets, env files, backups Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Data disappears after recreation. Check whether data was written to the container writable layer instead of a persistent volume.

**Commands / checks:** ``docker volume inspect`; `docker compose config`; inspect mount declarations`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q39. A Compose deployment needs a one-off migration job. How would you model its lifecycle and ensure it does not rerun unexpectedly?

**Difficulty:** Intermediate

**Interview-ready answer:** Treat schema migration and stateful upgrades as a separate controlled step. Verify backups, compatibility, and rollback/forward-fix paths before replacing containers. Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Volumes, bind mounts, secrets, env files, backups Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Treat schema migration and stateful upgrades as a separate controlled step. Verify backups, compatibility, and rollback/forward-fix paths before replacing containers.

**Production scenario:** Data disappears after recreation. Check whether data was written to the container writable layer instead of a persistent volume.

**Commands / checks:** ``docker volume inspect`; `docker compose config`; inspect mount declarations`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q40. How would you capture reproducible diagnostics from a failing Compose stack while redacting secrets?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Volumes, bind mounts, secrets, env files, backups Use named volumes for managed persistent data, bind mounts for deliberate host-path access, and secret mechanisms appropriate to the deployment. Back up and restore application-consistent data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** Data disappears after recreation. Check whether data was written to the container writable layer instead of a persistent volume.

**Commands / checks:** ``docker volume inspect`; `docker compose config`; inspect mount declarations`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q41. A new image passes tests but production still runs the old image. What image/tag/pull-policy and deployment checks do you make?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Health dependencies, profiles, jobs, rollout and environment separation Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** Dependent app starts before database is ready. Use health checks and supported dependency conditions, plus application-level retry logic.

**Commands / checks:** ``docker compose config`; `docker compose ps`; `docker compose logs -f`; `docker compose up -d``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q42. A container can write to a mounted directory but cannot rename files. Which filesystem, permissions, and mount options might explain it?

**Difficulty:** Intermediate

**Interview-ready answer:** Run as a non-root UID where possible, grant only required capabilities, use read-only root filesystems where compatible, and set explicit volume ownership and writable paths. Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Health dependencies, profiles, jobs, rollout and environment separation Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Run as a non-root UID where possible, grant only required capabilities, use read-only root filesystems where compatible, and set explicit volume ownership and writable paths.

**Production scenario:** Dependent app starts before database is ready. Use health checks and supported dependency conditions, plus application-level retry logic.

**Commands / checks:** ``docker compose config`; `docker compose ps`; `docker compose logs -f`; `docker compose up -d``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q43. How do you build an image with a private package token without baking the token into layers or build logs?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Health dependencies, profiles, jobs, rollout and environment separation Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** Dependent app starts before database is ready. Use health checks and supported dependency conditions, plus application-level retry logic.

**Commands / checks:** ``docker compose config`; `docker compose ps`; `docker compose logs -f`; `docker compose up -d``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q44. A Compose stack becomes unstable when all services restart at once. How would you manage startup order, resource pressure, and retries?

**Difficulty:** Intermediate

**Interview-ready answer:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior. Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Health dependencies, profiles, jobs, rollout and environment separation Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior.

**Production scenario:** Dependent app starts before database is ready. Use health checks and supported dependency conditions, plus application-level retry logic.

**Commands / checks:** ``docker compose config`; `docker compose ps`; `docker compose logs -f`; `docker compose up -d``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q45. An image build works only because an untracked local file exists in the build context. How do you detect and eliminate this dependency?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Health dependencies, profiles, jobs, rollout and environment separation Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** Dependent app starts before database is ready. Use health checks and supported dependency conditions, plus application-level retry logic.

**Commands / checks:** ``docker compose config`; `docker compose ps`; `docker compose logs -f`; `docker compose up -d``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q46. How would you generate and retain an SBOM for a Docker image and tie it to the exact deployed digest?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Health dependencies, profiles, jobs, rollout and environment separation Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** Dependent app starts before database is ready. Use health checks and supported dependency conditions, plus application-level retry logic.

**Commands / checks:** ``docker compose config`; `docker compose ps`; `docker compose logs -f`; `docker compose up -d``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q47. A service has a health check but Compose still starts a dependent task too early. What limitations and application-level retry behavior matter?

**Difficulty:** Intermediate

**Interview-ready answer:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior. Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Health dependencies, profiles, jobs, rollout and environment separation Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** A running container is not necessarily a ready application. Add a meaningful health check, bounded retries, correct signal handling, and application-level dependency retry behavior.

**Production scenario:** Dependent app starts before database is ready. Use health checks and supported dependency conditions, plus application-level retry logic.

**Commands / checks:** ``docker compose config`; `docker compose ps`; `docker compose logs -f`; `docker compose up -d``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q48. A container exposes an admin port accidentally. What checks and policy would prevent this from reaching a public host?

**Difficulty:** Advanced

**Interview-ready answer:** Compare the process's bind address, container port, published host port, host firewall, and network mode. A port mapping cannot make an app listening only on the wrong interface reachable. Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Health dependencies, profiles, jobs, rollout and environment separation Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Compare the process's bind address, container port, published host port, host firewall, and network mode. A port mapping cannot make an app listening only on the wrong interface reachable.

**Production scenario:** Dependent app starts before database is ready. Use health checks and supported dependency conditions, plus application-level retry logic.

**Commands / checks:** ``docker compose config`; `docker compose ps`; `docker compose logs -f`; `docker compose up -d``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q49. How do you safely update a stateful Compose application with a new image and schema migration?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers. Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Health dependencies, profiles, jobs, rollout and environment separation Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect image layers and build context. Use multi-stage builds, `.dockerignore`, pinned base images, and BuildKit secret mounts; deleting a secret in a later layer does not remove it from earlier layers.

**Production scenario:** Dependent app starts before database is ready. Use health checks and supported dependency conditions, plus application-level retry logic.

**Commands / checks:** ``docker compose config`; `docker compose ps`; `docker compose logs -f`; `docker compose up -d``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q50. Describe your troubleshooting order when an entire Compose stack becomes unreachable after a Docker Engine update.

**Difficulty:** Advanced

**Interview-ready answer:** Compare the process's bind address, container port, published host port, host firewall, and network mode. A port mapping cannot make an app listening only on the wrong interface reachable. Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Health dependencies, profiles, jobs, rollout and environment separation Compose is useful for multi-container application environments but is not a complete multi-node orchestrator. Define health checks, explicit networks, and reproducible configuration. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Compare the process's bind address, container port, published host port, host firewall, and network mode. A port mapping cannot make an app listening only on the wrong interface reachable.

**Production scenario:** Dependent app starts before database is ready. Use health checks and supported dependency conditions, plus application-level retry logic.

**Commands / checks:** ``docker compose config`; `docker compose ps`; `docker compose logs -f`; `docker compose up -d``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

## Official references

- [Dockerfile best practices](https://docs.docker.com/build/building/best-practices/)
- [Docker Engine security](https://docs.docker.com/engine/security/)
- [Compose releases](https://github.com/docker/compose/releases)

## Version and safety note

This guide is dated 18 October 2026. Release status, feature maturity, cloud-region availability, plugins, and supported versions change. Check the linked official documentation against the exact versions and architecture you use before applying any production recommendation.
