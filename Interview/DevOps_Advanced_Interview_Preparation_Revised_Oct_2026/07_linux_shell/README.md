# Linux & Shell Scripting — 50 Advanced Interview Questions

**Audience:** Mid-level/L2 DevOps Engineer (3–5 years).
**Reference date:** 18 October 2026.

**Focus:** Linux & Shell Scripting production operations, troubleshooting, security, and reliability

## How to practise
- First answer the **interview-ready answer** aloud in 60–90 seconds.
- Then explain the technical reasoning, trade-offs, and evidence you would collect.
- Treat commands as diagnostic starting points; replace placeholders and validate permissions.
- Do not run destructive or mutating commands in production without an approved change plan.
- Use the follow-up to test whether you understand the failure mechanism rather than memorizing a script.

## Questions

### Q1. A server has load average 20 but CPU utilization is only 15%. What does that suggest and what commands would you run?

**Difficulty:** Intermediate

**Interview-ready answer:** Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Compare run queue, CPU steal, I/O wait, blocked tasks, and device latency. Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Process state, signals, units, dependencies, logs, restart behavior Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Compare run queue, CPU steal, I/O wait, blocked tasks, and device latency.

**Production scenario:** Service repeatedly restarts. Inspect exit status, journal, environment, permissions, dependencies, and restart policy before changing the unit.

**Commands / checks:** ``systemctl status`; `journalctl -u <service> -b`; `ps auxf`; `systemctl cat``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q2. `df -h` shows 100% usage but `du` reports far less data. How do you investigate deleted-open files, mounts, and reserved blocks?

**Difficulty:** Intermediate

**Interview-ready answer:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data. Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Process state, signals, units, dependencies, logs, restart behavior Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data.

**Production scenario:** Service repeatedly restarts. Inspect exit status, journal, environment, permissions, dependencies, and restart policy before changing the unit.

**Commands / checks:** ``systemctl status`; `journalctl -u <service> -b`; `ps auxf`; `systemctl cat``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q3. A systemd service restarts repeatedly after a deployment. How do you find the original failure rather than just the restart symptom?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause. Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Process state, signals, units, dependencies, logs, restart behavior Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause.

**Production scenario:** Service repeatedly restarts. Inspect exit status, journal, environment, permissions, dependencies, and restart policy before changing the unit.

**Commands / checks:** ``systemctl status`; `journalctl -u <service> -b`; `ps auxf`; `systemctl cat``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q4. A process is in D state and cannot be killed. What does that state usually mean and which storage/kernel signals matter?

**Difficulty:** Intermediate

**Interview-ready answer:** D state usually means uninterruptible sleep, often while waiting on I/O or kernel work. Inspect storage latency, kernel logs, mount/NFS health, and do not assume SIGKILL will immediately remove it. Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Process state, signals, units, dependencies, logs, restart behavior Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** D state usually means uninterruptible sleep, often while waiting on I/O or kernel work. Inspect storage latency, kernel logs, mount/NFS health, and do not assume SIGKILL will immediately remove it.

**Production scenario:** Service repeatedly restarts. Inspect exit status, journal, environment, permissions, dependencies, and restart policy before changing the unit.

**Commands / checks:** ``systemctl status`; `journalctl -u <service> -b`; `ps auxf`; `systemctl cat``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q5. A shell pipeline returns exit code zero even though an earlier command failed. How do you correct the script and test it?

**Difficulty:** Advanced

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Process state, signals, units, dependencies, logs, restart behavior Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Service repeatedly restarts. Inspect exit status, journal, environment, permissions, dependencies, and restart policy before changing the unit.

**Commands / checks:** ``systemctl status`; `journalctl -u <service> -b`; `ps auxf`; `systemctl cat``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q6. A script deletes a directory built from a variable. How do you prevent empty-variable and path-expansion disasters?

**Difficulty:** Intermediate

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Process state, signals, units, dependencies, logs, restart behavior Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Service repeatedly restarts. Inspect exit status, journal, environment, permissions, dependencies, and restart policy before changing the unit.

**Commands / checks:** ``systemctl status`; `journalctl -u <service> -b`; `ps auxf`; `systemctl cat``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q7. A server cannot resolve DNS but can connect to known IP addresses. What resolver and NSS checks do you perform?

**Difficulty:** Intermediate

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Process state, signals, units, dependencies, logs, restart behavior Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Service repeatedly restarts. Inspect exit status, journal, environment, permissions, dependencies, and restart policy before changing the unit.

**Commands / checks:** ``systemctl status`; `journalctl -u <service> -b`; `ps auxf`; `systemctl cat``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q8. A process is listening on 127.0.0.1:8080 but remote clients cannot reach it. What is the likely bind-address issue?

**Difficulty:** Advanced

**Interview-ready answer:** Check listener bind address, local socket state, route, host firewall, cloud firewall, return path, and packet evidence. Test each layer separately. Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Process state, signals, units, dependencies, logs, restart behavior Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Check listener bind address, local socket state, route, host firewall, cloud firewall, return path, and packet evidence. Test each layer separately.

**Production scenario:** Service repeatedly restarts. Inspect exit status, journal, environment, permissions, dependencies, and restart policy before changing the unit.

**Commands / checks:** ``systemctl status`; `journalctl -u <service> -b`; `ps auxf`; `systemctl cat``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q9. Memory usage rises over several hours and the kernel OOM-kills the application. How do you distinguish leak, cache, and workload pressure?

**Difficulty:** Intermediate

**Interview-ready answer:** D state usually means uninterruptible sleep, often while waiting on I/O or kernel work. Inspect storage latency, kernel logs, mount/NFS health, and do not assume SIGKILL will immediately remove it. Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Process state, signals, units, dependencies, logs, restart behavior Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** D state usually means uninterruptible sleep, often while waiting on I/O or kernel work. Inspect storage latency, kernel logs, mount/NFS health, and do not assume SIGKILL will immediately remove it.

**Production scenario:** Service repeatedly restarts. Inspect exit status, journal, environment, permissions, dependencies, and restart policy before changing the unit.

**Commands / checks:** ``systemctl status`; `journalctl -u <service> -b`; `ps auxf`; `systemctl cat``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q10. SSH access fails after a firewall change. How do you recover access and validate the return path safely?

**Difficulty:** Advanced

**Interview-ready answer:** Check listener bind address, local socket state, route, host firewall, cloud firewall, return path, and packet evidence. Test each layer separately. Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Process state, signals, units, dependencies, logs, restart behavior Start with service status and logs, then trace process, dependency, permission, and resource issues. Use systemd unit semantics deliberately and avoid blind restart loops. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Check listener bind address, local socket state, route, host firewall, cloud firewall, return path, and packet evidence. Test each layer separately.

**Production scenario:** Service repeatedly restarts. Inspect exit status, journal, environment, permissions, dependencies, and restart policy before changing the unit.

**Commands / checks:** ``systemctl status`; `journalctl -u <service> -b`; `ps auxf`; `systemctl cat``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q11. A filesystem has free space but file creation fails. How do you check inode exhaustion and filesystem health?

**Difficulty:** Intermediate

**Interview-ready answer:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data. Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Load average, CPU saturation, memory pressure, OOM, I/O wait Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data.

**Production scenario:** Load is high but CPU looks idle. Inspect blocked I/O, D-state processes, storage latency, and kernel logs.

**Commands / checks:** ``top`; `vmstat 1`; `free -h`; `iostat -xz 1`; `dmesg -T``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q12. A service works manually but fails under systemd. Which environment, working directory, user, limits, and permission differences do you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause. Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Load average, CPU saturation, memory pressure, OOM, I/O wait Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause.

**Production scenario:** Load is high but CPU looks idle. Inspect blocked I/O, D-state processes, storage latency, and kernel logs.

**Commands / checks:** ``top`; `vmstat 1`; `free -h`; `iostat -xz 1`; `dmesg -T``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q13. A log file grows rapidly and fills the root filesystem. How do you find the source and rotate logs without losing evidence?

**Difficulty:** Advanced

**Interview-ready answer:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data. Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Load average, CPU saturation, memory pressure, OOM, I/O wait Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data.

**Production scenario:** Load is high but CPU looks idle. Inspect blocked I/O, D-state processes, storage latency, and kernel logs.

**Commands / checks:** ``top`; `vmstat 1`; `free -h`; `iostat -xz 1`; `dmesg -T``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q14. An application is slow but CPU and memory appear normal. How would you use `vmstat`, `iostat`, `ss`, and process state to narrow it down?

**Difficulty:** Intermediate

**Interview-ready answer:** Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Compare run queue, CPU steal, I/O wait, blocked tasks, and device latency. Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Load average, CPU saturation, memory pressure, OOM, I/O wait Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Compare run queue, CPU steal, I/O wait, blocked tasks, and device latency.

**Production scenario:** Load is high but CPU looks idle. Inspect blocked I/O, D-state processes, storage latency, and kernel logs.

**Commands / checks:** ``top`; `vmstat 1`; `free -h`; `iostat -xz 1`; `dmesg -T``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q15. A shell script processes filenames containing spaces and newlines incorrectly. How do you make iteration safe?

**Difficulty:** Advanced

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Load average, CPU saturation, memory pressure, OOM, I/O wait Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Load is high but CPU looks idle. Inspect blocked I/O, D-state processes, storage latency, and kernel logs.

**Commands / checks:** ``top`; `vmstat 1`; `free -h`; `iostat -xz 1`; `dmesg -T``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q16. A cron job runs with a different PATH and locale than an interactive shell. How do you make its environment explicit?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause. Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Load average, CPU saturation, memory pressure, OOM, I/O wait Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause.

**Production scenario:** Load is high but CPU looks idle. Inspect blocked I/O, D-state processes, storage latency, and kernel logs.

**Commands / checks:** ``top`; `vmstat 1`; `free -h`; `iostat -xz 1`; `dmesg -T``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q17. A TCP port is reachable locally but not from another subnet. How do you trace routes, firewall rules, and listening addresses?

**Difficulty:** Intermediate

**Interview-ready answer:** Check listener bind address, local socket state, route, host firewall, cloud firewall, return path, and packet evidence. Test each layer separately. Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Load average, CPU saturation, memory pressure, OOM, I/O wait Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Check listener bind address, local socket state, route, host firewall, cloud firewall, return path, and packet evidence. Test each layer separately.

**Production scenario:** Load is high but CPU looks idle. Inspect blocked I/O, D-state processes, storage latency, and kernel logs.

**Commands / checks:** ``top`; `vmstat 1`; `free -h`; `iostat -xz 1`; `dmesg -T``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q18. A package update changed a shared library and a service no longer starts. What logs and dependency checks do you use?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause. Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Load average, CPU saturation, memory pressure, OOM, I/O wait Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause.

**Production scenario:** Load is high but CPU looks idle. Inspect blocked I/O, D-state processes, storage latency, and kernel logs.

**Commands / checks:** ``top`; `vmstat 1`; `free -h`; `iostat -xz 1`; `dmesg -T``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q19. A system clock is skewed and TLS validation fails. How do you verify time synchronization and certificate validity?

**Difficulty:** Intermediate

**Interview-ready answer:** Check system time, hostname/SAN, certificate chain, CA bundle, expiry, and proxy interception. Compare the certificate actually served with the expected one. Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Load average, CPU saturation, memory pressure, OOM, I/O wait Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Check system time, hostname/SAN, certificate chain, CA bundle, expiry, and proxy interception. Compare the certificate actually served with the expected one.

**Production scenario:** Load is high but CPU looks idle. Inspect blocked I/O, D-state processes, storage latency, and kernel logs.

**Commands / checks:** ``top`; `vmstat 1`; `free -h`; `iostat -xz 1`; `dmesg -T``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q20. A script retries a failed operation indefinitely. How do you add bounded retries, backoff, and meaningful exit status?

**Difficulty:** Advanced

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Load average, CPU saturation, memory pressure, OOM, I/O wait Use multiple signals to distinguish CPU saturation, memory pressure, and storage latency. Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Load is high but CPU looks idle. Inspect blocked I/O, D-state processes, storage latency, and kernel logs.

**Commands / checks:** ``top`; `vmstat 1`; `free -h`; `iostat -xz 1`; `dmesg -T``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q21. A process consumes file descriptors until new connections fail. How do you confirm descriptor exhaustion and identify the leak?

**Difficulty:** Intermediate

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Capacity, inodes, mounts, deleted-open files, permissions Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** `df` is full but `du` is small. Look for deleted open files, hidden mount contents, reserved blocks, or filesystem metadata.

**Commands / checks:** ``df -h`; `df -i`; `du -xhd1`; `lsof +L1`; `findmnt``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q22. A disk latency alert fires while throughput is low. What do I/O wait, queue depth, and per-device latency tell you?

**Difficulty:** Intermediate

**Interview-ready answer:** Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Compare run queue, CPU steal, I/O wait, blocked tasks, and device latency. Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Capacity, inodes, mounts, deleted-open files, permissions Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Compare run queue, CPU steal, I/O wait, blocked tasks, and device latency.

**Production scenario:** `df` is full but `du` is small. Look for deleted open files, hidden mount contents, reserved blocks, or filesystem metadata.

**Commands / checks:** ``df -h`; `df -i`; `du -xhd1`; `lsof +L1`; `findmnt``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q23. A systemd unit has `After=network-online.target` but starts before an external dependency is usable. Why can this happen?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause. Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Capacity, inodes, mounts, deleted-open files, permissions Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause.

**Production scenario:** `df` is full but `du` is small. Look for deleted open files, hidden mount contents, reserved blocks, or filesystem metadata.

**Commands / checks:** ``df -h`; `df -i`; `du -xhd1`; `lsof +L1`; `findmnt``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q24. A bash script uses `set -e` but still continues after a failure in a conditional/pipeline context. What are the limitations?

**Difficulty:** Intermediate

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Capacity, inodes, mounts, deleted-open files, permissions Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** `df` is full but `du` is small. Look for deleted open files, hidden mount contents, reserved blocks, or filesystem metadata.

**Commands / checks:** ``df -h`; `df -i`; `du -xhd1`; `lsof +L1`; `findmnt``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q25. A service is killed during deployment and corrupts a file. How should signal handling and graceful shutdown work?

**Difficulty:** Advanced

**Interview-ready answer:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data. Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Capacity, inodes, mounts, deleted-open files, permissions Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data.

**Production scenario:** `df` is full but `du` is small. Look for deleted open files, hidden mount contents, reserved blocks, or filesystem metadata.

**Commands / checks:** ``df -h`; `df -i`; `du -xhd1`; `lsof +L1`; `findmnt``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q26. A server reports many TCP connections in TIME_WAIT. How do you decide whether this is normal or an actual port/resource issue?

**Difficulty:** Intermediate

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Capacity, inodes, mounts, deleted-open files, permissions Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** `df` is full but `du` is small. Look for deleted open files, hidden mount contents, reserved blocks, or filesystem metadata.

**Commands / checks:** ``df -h`; `df -i`; `du -xhd1`; `lsof +L1`; `findmnt``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q27. A mounted filesystem disappears after reboot. How do you inspect `/etc/fstab`, systemd mount dependencies, and device identifiers?

**Difficulty:** Intermediate

**Interview-ready answer:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data. Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Capacity, inodes, mounts, deleted-open files, permissions Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data.

**Production scenario:** `df` is full but `du` is small. Look for deleted open files, hidden mount contents, reserved blocks, or filesystem metadata.

**Commands / checks:** ``df -h`; `df -i`; `du -xhd1`; `lsof +L1`; `findmnt``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q28. A log contains credentials accidentally. What containment, rotation, access restriction, and code changes are needed?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Capacity, inodes, mounts, deleted-open files, permissions Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** `df` is full but `du` is small. Look for deleted open files, hidden mount contents, reserved blocks, or filesystem metadata.

**Commands / checks:** ``df -h`; `df -i`; `du -xhd1`; `lsof +L1`; `findmnt``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q29. A Linux host shows unexpected outbound connections. How would you preserve evidence and identify owning processes without destroying state?

**Difficulty:** Intermediate

**Interview-ready answer:** Correlate process RSS, cgroup limits, kernel OOM logs, cache/slab behavior, and workload changes. Distinguish a leak from a legitimate workload increase before changing limits. Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Capacity, inodes, mounts, deleted-open files, permissions Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Correlate process RSS, cgroup limits, kernel OOM logs, cache/slab behavior, and workload changes. Distinguish a leak from a legitimate workload increase before changing limits.

**Production scenario:** `df` is full but `du` is small. Look for deleted open files, hidden mount contents, reserved blocks, or filesystem metadata.

**Commands / checks:** ``df -h`; `df -i`; `du -xhd1`; `lsof +L1`; `findmnt``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q30. How would you make a shell script safe to rerun after partial failure?

**Difficulty:** Advanced

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Capacity, inodes, mounts, deleted-open files, permissions Check both block capacity and inode exhaustion, verify mounts, and identify large files or deleted-but-open files before deleting data. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** `df` is full but `du` is small. Look for deleted open files, hidden mount contents, reserved blocks, or filesystem metadata.

**Commands / checks:** ``df -h`; `df -i`; `du -xhd1`; `lsof +L1`; `findmnt``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q31. A service starts before a required mount is available and writes data to the underlying root directory. How do you prevent recurrence?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause. Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Quoting, exit codes, pipelines, traps, signals, idempotence Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause.

**Production scenario:** Pipeline masks a failing command. Use `pipefail`, inspect `PIPESTATUS`, and avoid assuming `set -e` catches every failure context.

**Commands / checks:** ``set -Eeuo pipefail` where appropriate; `shellcheck`; `bash -x` in safe contexts`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q32. A user can read a file through group membership they should not have. How do you inspect effective permissions and ACLs?

**Difficulty:** Intermediate

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Quoting, exit codes, pipelines, traps, signals, idempotence Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Pipeline masks a failing command. Use `pipefail`, inspect `PIPESTATUS`, and avoid assuming `set -e` catches every failure context.

**Commands / checks:** ``set -Eeuo pipefail` where appropriate; `shellcheck`; `bash -x` in safe contexts`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q33. An application reports `Too many open files` only under load. How do you inspect per-process limits and system limits?

**Difficulty:** Advanced

**Interview-ready answer:** Check listener bind address, local socket state, route, host firewall, cloud firewall, return path, and packet evidence. Test each layer separately. Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Quoting, exit codes, pipelines, traps, signals, idempotence Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Check listener bind address, local socket state, route, host firewall, cloud firewall, return path, and packet evidence. Test each layer separately.

**Production scenario:** Pipeline masks a failing command. Use `pipefail`, inspect `PIPESTATUS`, and avoid assuming `set -e` catches every failure context.

**Commands / checks:** ``set -Eeuo pipefail` where appropriate; `shellcheck`; `bash -x` in safe contexts`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q34. A deployment script uses `curl` without checking HTTP status. How can it report success after an HTTP 500 response?

**Difficulty:** Intermediate

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Quoting, exit codes, pipelines, traps, signals, idempotence Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Pipeline masks a failing command. Use `pipefail`, inspect `PIPESTATUS`, and avoid assuming `set -e` catches every failure context.

**Commands / checks:** ``set -Eeuo pipefail` where appropriate; `shellcheck`; `bash -x` in safe contexts`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q35. A server is out of memory but the top process list does not explain it. What cgroup, kernel, and container evidence would you check?

**Difficulty:** Advanced

**Interview-ready answer:** D state usually means uninterruptible sleep, often while waiting on I/O or kernel work. Inspect storage latency, kernel logs, mount/NFS health, and do not assume SIGKILL will immediately remove it. Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Quoting, exit codes, pipelines, traps, signals, idempotence Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** D state usually means uninterruptible sleep, often while waiting on I/O or kernel work. Inspect storage latency, kernel logs, mount/NFS health, and do not assume SIGKILL will immediately remove it.

**Production scenario:** Pipeline masks a failing command. Use `pipefail`, inspect `PIPESTATUS`, and avoid assuming `set -e` catches every failure context.

**Commands / checks:** ``set -Eeuo pipefail` where appropriate; `shellcheck`; `bash -x` in safe contexts`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q36. A DNS lookup works with `dig` but an application cannot resolve the same name. What NSS, search-domain, and runtime differences matter?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare resolver configuration, NSS order, search domains, `/etc/hosts`, caching services, and application runtime. `dig` tests DNS directly and may not reproduce libc name resolution. Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Quoting, exit codes, pipelines, traps, signals, idempotence Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Compare resolver configuration, NSS order, search domains, `/etc/hosts`, caching services, and application runtime. `dig` tests DNS directly and may not reproduce libc name resolution.

**Production scenario:** Pipeline masks a failing command. Use `pipefail`, inspect `PIPESTATUS`, and avoid assuming `set -e` catches every failure context.

**Commands / checks:** ``set -Eeuo pipefail` where appropriate; `shellcheck`; `bash -x` in safe contexts`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q37. A TLS connection fails with `certificate verify failed`. How do you inspect CA trust, hostname, intermediate chain, and clock?

**Difficulty:** Intermediate

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Quoting, exit codes, pipelines, traps, signals, idempotence Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Pipeline masks a failing command. Use `pipefail`, inspect `PIPESTATUS`, and avoid assuming `set -e` catches every failure context.

**Commands / checks:** ``set -Eeuo pipefail` where appropriate; `shellcheck`; `bash -x` in safe contexts`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q38. A systemd service has the wrong restart policy and creates a crash loop. What policy and rate-limit settings should be reviewed?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause. Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Quoting, exit codes, pipelines, traps, signals, idempotence Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause.

**Production scenario:** Pipeline masks a failing command. Use `pipefail`, inspect `PIPESTATUS`, and avoid assuming `set -e` catches every failure context.

**Commands / checks:** ``set -Eeuo pipefail` where appropriate; `shellcheck`; `bash -x` in safe contexts`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q39. A cleanup script follows symlinks and deletes files outside its target directory. How would you redesign the safety checks?

**Difficulty:** Intermediate

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Quoting, exit codes, pipelines, traps, signals, idempotence Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Pipeline masks a failing command. Use `pipefail`, inspect `PIPESTATUS`, and avoid assuming `set -e` catches every failure context.

**Commands / checks:** ``set -Eeuo pipefail` where appropriate; `shellcheck`; `bash -x` in safe contexts`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q40. How do you write a health-check script that returns meaningful exit codes and does not hang indefinitely?

**Difficulty:** Advanced

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Quoting, exit codes, pipelines, traps, signals, idempotence Write scripts with strict but understood error handling, quote expansions, validate inputs, use traps for cleanup, and make retries bounded and safe. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Pipeline masks a failing command. Use `pipefail`, inspect `PIPESTATUS`, and avoid assuming `set -e` catches every failure context.

**Commands / checks:** ``set -Eeuo pipefail` where appropriate; `shellcheck`; `bash -x` in safe contexts`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q41. A kernel upgrade causes a network interface to disappear. What logs, modules, device naming, and rollback evidence do you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data. Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Sockets, routes, DNS, firewall, SSH, certificates and permissions Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data.

**Production scenario:** Port is listening locally but remote clients fail. Check bind address, firewall, route, cloud security rules, NAT, and return path.

**Commands / checks:** ``ss -lntp`; `ip route`; `getent hosts`; `curl -v`; `openssl s_client``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q42. An app is slow only on one VM of the same size. How do you compare noisy-neighbor effects, CPU steal, disk latency, and configuration drift?

**Difficulty:** Intermediate

**Interview-ready answer:** Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Compare run queue, CPU steal, I/O wait, blocked tasks, and device latency. Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Sockets, routes, DNS, firewall, SSH, certificates and permissions Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Compare run queue, CPU steal, I/O wait, blocked tasks, and device latency.

**Production scenario:** Port is listening locally but remote clients fail. Check bind address, firewall, route, cloud security rules, NAT, and return path.

**Commands / checks:** ``ss -lntp`; `ip route`; `getent hosts`; `curl -v`; `openssl s_client``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q43. A command works in Bash but fails when invoked by `/bin/sh`. How do you make scripts portable or explicitly require Bash?

**Difficulty:** Advanced

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Sockets, routes, DNS, firewall, SSH, certificates and permissions Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Port is listening locally but remote clients fail. Check bind address, firewall, route, cloud security rules, NAT, and return path.

**Commands / checks:** ``ss -lntp`; `ip route`; `getent hosts`; `curl -v`; `openssl s_client``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q44. A production host has a full `/var` partition but root has free space. What filesystem and mount layout facts matter?

**Difficulty:** Intermediate

**Interview-ready answer:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data. Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Sockets, routes, DNS, firewall, SSH, certificates and permissions Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data.

**Production scenario:** Port is listening locally but remote clients fail. Check bind address, firewall, route, cloud security rules, NAT, and return path.

**Commands / checks:** ``ss -lntp`; `ip route`; `getent hosts`; `curl -v`; `openssl s_client``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q45. An SSH connection hangs after authentication. How would you separate shell startup, PAM, home-directory, and network issues?

**Difficulty:** Advanced

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Sockets, routes, DNS, firewall, SSH, certificates and permissions Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Port is listening locally but remote clients fail. Check bind address, firewall, route, cloud security rules, NAT, and return path.

**Commands / checks:** ``ss -lntp`; `ip route`; `getent hosts`; `curl -v`; `openssl s_client``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q46. A systemd service can write logs but cannot write its data directory. How do user, group, umask, ACL, and SELinux/AppArmor rules factor in?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause. Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Sockets, routes, DNS, firewall, SSH, certificates and permissions Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect the first failure in the journal, unit environment, working directory, user/group, dependencies, restart policy, and resource limits. Repeated restarts are symptoms, not root cause.

**Production scenario:** Port is listening locally but remote clients fail. Check bind address, firewall, route, cloud security rules, NAT, and return path.

**Commands / checks:** ``ss -lntp`; `ip route`; `getent hosts`; `curl -v`; `openssl s_client``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q47. A shell script uses unquoted command substitution to build an `rm` command. What safer patterns should replace it?

**Difficulty:** Intermediate

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Sockets, routes, DNS, firewall, SSH, certificates and permissions Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Port is listening locally but remote clients fail. Check bind address, firewall, route, cloud security rules, NAT, and return path.

**Commands / checks:** ``ss -lntp`; `ip route`; `getent hosts`; `curl -v`; `openssl s_client``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q48. How do you investigate intermittent packet loss from Linux without assuming the network team is at fault?

**Difficulty:** Advanced

**Interview-ready answer:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions. Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Sockets, routes, DNS, firewall, SSH, certificates and permissions Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Quote expansions, validate paths and inputs, use explicit exit-code handling and `pipefail` where appropriate, and add traps/cleanup for partial failure. `set -e` has contextual exceptions.

**Production scenario:** Port is listening locally but remote clients fail. Check bind address, firewall, route, cloud security rules, NAT, and return path.

**Commands / checks:** ``ss -lntp`; `ip route`; `getent hosts`; `curl -v`; `openssl s_client``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q49. A process has high CPU but low throughput. What profiling, lock contention, syscall, and throttling evidence would you gather?

**Difficulty:** Intermediate

**Interview-ready answer:** Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Compare run queue, CPU steal, I/O wait, blocked tasks, and device latency. Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Sockets, routes, DNS, firewall, SSH, certificates and permissions Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Load average includes runnable and uninterruptible tasks; it is not CPU utilization alone. Compare run queue, CPU steal, I/O wait, blocked tasks, and device latency.

**Production scenario:** Port is listening locally but remote clients fail. Check bind address, firewall, route, cloud security rules, NAT, and return path.

**Commands / checks:** ``ss -lntp`; `ip route`; `getent hosts`; `curl -v`; `openssl s_client``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q50. Describe a safe incident workflow for a full production filesystem when the application must remain online.

**Difficulty:** Advanced

**Interview-ready answer:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data. Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Sockets, routes, DNS, firewall, SSH, certificates and permissions Test each layer independently: name resolution, route, TCP reachability, TLS handshake, application protocol, and authorization. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Check both block capacity and inode availability, then inspect deleted-open files, mount points, reserved blocks, and filesystem metadata before deleting data.

**Production scenario:** Port is listening locally but remote clients fail. Check bind address, firewall, route, cloud security rules, NAT, and return path.

**Commands / checks:** ``ss -lntp`; `ip route`; `getent hosts`; `curl -v`; `openssl s_client``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

## Official references

- [Bash manual](https://www.gnu.org/software/bash/manual/)
- [ShellCheck](https://www.shellcheck.net/)
- [systemd manual](https://www.freedesktop.org/software/systemd/man/)

## Version and safety note

This guide is dated 18 October 2026. Release status, feature maturity, cloud-region availability, plugins, and supported versions change. Check the linked official documentation against the exact versions and architecture you use before applying any production recommendation.
