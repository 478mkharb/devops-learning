# Networking, DNS, Load Balancers & TLS — 50 Advanced Interview Questions

**Audience:** Mid-level/L2 DevOps Engineer (3–5 years).
**Reference date:** 18 October 2026.

**Focus:** Networking, DNS, Load Balancers & TLS production operations, troubleshooting, security, and reliability

## How to practise
- First answer the **interview-ready answer** aloud in 60–90 seconds.
- Then explain the technical reasoning, trade-offs, and evidence you would collect.
- Treat commands as diagnostic starting points; replace placeholders and validate permissions.
- Do not run destructive or mutating commands in production without an approved change plan.
- Use the follow-up to test whether you understand the failure mechanism rather than memorizing a script.

## Questions

### Q1. A service resolves to an old IP for only some clients after a DNS change. How do TTL, recursive caches, split-horizon zones, and local caches affect this?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, MTU, routes, NAT, ports, packet loss and timeouts Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Connections hang only for large payloads. Investigate MTU/path-MTU discovery, fragmentation, MSS, and middleboxes.

**Commands / checks:** ``ip route`; `ss -tnp`; `tcpdump`; `traceroute`/`mtr` where permitted`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q2. An ALB returns 503 while targets are registered. What target-health, listener, port, and application checks do you perform?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, MTU, routes, NAT, ports, packet loss and timeouts Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Connections hang only for large payloads. Investigate MTU/path-MTU discovery, fragmentation, MSS, and middleboxes.

**Commands / checks:** ``ip route`; `ss -tnp`; `tcpdump`; `traceroute`/`mtr` where permitted`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q3. A TLS certificate works in a browser but fails inside a minimal container. How do CA bundles, intermediate certificates, SAN, and clock skew matter?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, MTU, routes, NAT, ports, packet loss and timeouts Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Connections hang only for large payloads. Investigate MTU/path-MTU discovery, fragmentation, MSS, and middleboxes.

**Commands / checks:** ``ip route`; `ss -tnp`; `tcpdump`; `traceroute`/`mtr` where permitted`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q4. Small HTTP requests work but large uploads hang. How would you investigate MTU, MSS, path-MTU discovery, proxies, and timeouts?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare packet size, retransmissions, interface MTU, MSS, path-MTU discovery, and middleboxes. Small requests succeeding while large ones hang can indicate a path-MTU issue. Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, MTU, routes, NAT, ports, packet loss and timeouts Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Compare packet size, retransmissions, interface MTU, MSS, path-MTU discovery, and middleboxes. Small requests succeeding while large ones hang can indicate a path-MTU issue.

**Production scenario:** Connections hang only for large payloads. Investigate MTU/path-MTU discovery, fragmentation, MSS, and middleboxes.

**Commands / checks:** ``ip route`; `ss -tnp`; `tcpdump`; `traceroute`/`mtr` where permitted`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q5. A port listens on localhost but remote connections fail. What bind-address, route, and firewall checks do you run?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, MTU, routes, NAT, ports, packet loss and timeouts Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Connections hang only for large payloads. Investigate MTU/path-MTU discovery, fragmentation, MSS, and middleboxes.

**Commands / checks:** ``ip route`; `ss -tnp`; `tcpdump`; `traceroute`/`mtr` where permitted`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q6. Clients receive intermittent 502 errors after a deployment. How do you distinguish proxy-to-target connection failures from upstream application errors?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, MTU, routes, NAT, ports, packet loss and timeouts Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Connections hang only for large payloads. Investigate MTU/path-MTU discovery, fragmentation, MSS, and middleboxes.

**Commands / checks:** ``ip route`; `ss -tnp`; `tcpdump`; `traceroute`/`mtr` where permitted`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q7. DNS lookup works with `dig` but an application cannot resolve the same hostname. What NSS, resolver, search-domain, and runtime differences matter?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, MTU, routes, NAT, ports, packet loss and timeouts Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Connections hang only for large payloads. Investigate MTU/path-MTU discovery, fragmentation, MSS, and middleboxes.

**Commands / checks:** ``ip route`; `ss -tnp`; `tcpdump`; `traceroute`/`mtr` where permitted`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q8. A load balancer health check succeeds but users receive 500s. How should health checks and synthetic tests be designed?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, MTU, routes, NAT, ports, packet loss and timeouts Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Connections hang only for large payloads. Investigate MTU/path-MTU discovery, fragmentation, MSS, and middleboxes.

**Commands / checks:** ``ip route`; `ss -tnp`; `tcpdump`; `traceroute`/`mtr` where permitted`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q9. An mTLS client is rejected even though its certificate is unexpired. What chain, trust-store, EKU, SAN, and client-auth checks apply?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, MTU, routes, NAT, ports, packet loss and timeouts Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Connections hang only for large payloads. Investigate MTU/path-MTU discovery, fragmentation, MSS, and middleboxes.

**Commands / checks:** ``ip route`; `ss -tnp`; `tcpdump`; `traceroute`/`mtr` where permitted`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q10. Design a debugging sequence for a timeout that could originate in DNS, TCP, TLS, or the application.

**Difficulty:** Advanced

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, MTU, routes, NAT, ports, packet loss and timeouts Trace source and destination addressing, routing, firewall rules, TCP handshake, MTU, and return traffic. Distinguish connection timeout, reset, and application response. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Connections hang only for large payloads. Investigate MTU/path-MTU discovery, fragmentation, MSS, and middleboxes.

**Commands / checks:** ``ip route`; `ss -tnp`; `tcpdump`; `traceroute`/`mtr` where permitted`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q11. A certificate renewal completed, but clients still see the old certificate. What SNI, listener binding, proxy, and cache checks do you make?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Recursive resolution, TTL, records, split-horizon DNS, caching Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Some clients resolve old IPs. Compare recursive resolvers and TTLs, and check split-horizon records and client-side caches.

**Commands / checks:** ``dig A`; `dig +trace`; `getent hosts`; inspect resolver configuration`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q12. A service works from one subnet but not another. How do you trace routing, NACLs, security groups, NAT, and return traffic?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare A and AAAA answers, listener support, IPv6 routes, security policy, and client preference. Validate both address families independently. Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Recursive resolution, TTL, records, split-horizon DNS, caching Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Compare A and AAAA answers, listener support, IPv6 routes, security policy, and client preference. Validate both address families independently.

**Production scenario:** Some clients resolve old IPs. Compare recursive resolvers and TTLs, and check split-horizon records and client-side caches.

**Commands / checks:** ``dig A`; `dig +trace`; `getent hosts`; inspect resolver configuration`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q13. A DNS record has the correct value in the authoritative zone but clients still fail. What resolver path and caching evidence would you collect?

**Difficulty:** Advanced

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Recursive resolution, TTL, records, split-horizon DNS, caching Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Some clients resolve old IPs. Compare recursive resolvers and TTLs, and check split-horizon records and client-side caches.

**Commands / checks:** ``dig A`; `dig +trace`; `getent hosts`; inspect resolver configuration`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q14. An ingress/load balancer redirects HTTP to HTTPS in a loop. How do forwarded-proto headers and TLS termination explain the issue?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Recursive resolution, TTL, records, split-horizon DNS, caching Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Some clients resolve old IPs. Compare recursive resolvers and TTLs, and check split-horizon records and client-side caches.

**Commands / checks:** ``dig A`; `dig +trace`; `getent hosts`; inspect resolver configuration`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q15. A connection resets immediately rather than timing out. What does the difference suggest, and what packet/log evidence helps?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Recursive resolution, TTL, records, split-horizon DNS, caching Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Some clients resolve old IPs. Compare recursive resolvers and TTLs, and check split-horizon records and client-side caches.

**Commands / checks:** ``dig A`; `dig +trace`; `getent hosts`; inspect resolver configuration`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q16. A TLS handshake fails only for older clients. How do protocol versions, cipher suites, and certificate algorithms factor in?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Recursive resolution, TTL, records, split-horizon DNS, caching Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Some clients resolve old IPs. Compare recursive resolvers and TTLs, and check split-horizon records and client-side caches.

**Commands / checks:** ``dig A`; `dig +trace`; `getent hosts`; inspect resolver configuration`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q17. A load balancer sends traffic to a backend on the wrong port after a deployment. Which listener, target group, service, and health-check settings matter?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Recursive resolution, TTL, records, split-horizon DNS, caching Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Some clients resolve old IPs. Compare recursive resolvers and TTLs, and check split-horizon records and client-side caches.

**Commands / checks:** ``dig A`; `dig +trace`; `getent hosts`; inspect resolver configuration`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q18. How would you troubleshoot intermittent DNS SERVFAIL without assuming the record is missing?

**Difficulty:** Advanced

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Recursive resolution, TTL, records, split-horizon DNS, caching Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Some clients resolve old IPs. Compare recursive resolvers and TTLs, and check split-horizon records and client-side caches.

**Commands / checks:** ``dig A`; `dig +trace`; `getent hosts`; inspect resolver configuration`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q19. An application behind a reverse proxy records the proxy IP instead of the client IP. How should forwarded headers be trusted safely?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Recursive resolution, TTL, records, split-horizon DNS, caching Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Some clients resolve old IPs. Compare recursive resolvers and TTLs, and check split-horizon records and client-side caches.

**Commands / checks:** ``dig A`; `dig +trace`; `getent hosts`; inspect resolver configuration`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q20. A client reports a 504 but the application logs show no request. Which timeout and proxy layers would you investigate?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Recursive resolution, TTL, records, split-horizon DNS, caching Check authoritative records, resolver path, TTL, caching, search domains, and private/public zones. DNS changes may not be visible until caches expire. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Some clients resolve old IPs. Compare recursive resolvers and TTLs, and check split-horizon records and client-side caches.

**Commands / checks:** ``dig A`; `dig +trace`; `getent hosts`; inspect resolver configuration`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q21. A TCP connection succeeds but HTTPS fails. What additional checks distinguish TLS failure from application-layer failure?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** L4/L7, health checks, stickiness, X-Forwarded-For, timeouts Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Load balancer reports unhealthy targets. Check health path/status, target port, security rules, app bind address, and expected protocol.

**Commands / checks:** ``curl -v`; load balancer target health; application access logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q22. A DNS cutover must happen with minimal disruption. How would you plan TTL changes, validation, rollback, and resolver behavior?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** L4/L7, health checks, stickiness, X-Forwarded-For, timeouts Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Load balancer reports unhealthy targets. Check health path/status, target port, security rules, app bind address, and expected protocol.

**Commands / checks:** ``curl -v`; load balancer target health; application access logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q23. An API is slow only when TLS connections are newly created. What handshake, connection reuse, certificate validation, and CPU evidence would you inspect?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** L4/L7, health checks, stickiness, X-Forwarded-For, timeouts Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Load balancer reports unhealthy targets. Check health path/status, target port, security rules, app bind address, and expected protocol.

**Commands / checks:** ``curl -v`; load balancer target health; application access logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q24. A load balancer marks healthy targets unhealthy intermittently. How do thresholds, slow start, grace periods, and health path behavior matter?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** L4/L7, health checks, stickiness, X-Forwarded-For, timeouts Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Load balancer reports unhealthy targets. Check health path/status, target port, security rules, app bind address, and expected protocol.

**Commands / checks:** ``curl -v`; load balancer target health; application access logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q25. A firewall allows outbound traffic but responses never arrive. How do stateful rules, NACLs, NAT, and asymmetric routes differ?

**Difficulty:** Advanced

**Interview-ready answer:** Compare packet size, retransmissions, interface MTU, MSS, path-MTU discovery, and middleboxes. Small requests succeeding while large ones hang can indicate a path-MTU issue. Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** L4/L7, health checks, stickiness, X-Forwarded-For, timeouts Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Compare packet size, retransmissions, interface MTU, MSS, path-MTU discovery, and middleboxes. Small requests succeeding while large ones hang can indicate a path-MTU issue.

**Production scenario:** Load balancer reports unhealthy targets. Check health path/status, target port, security rules, app bind address, and expected protocol.

**Commands / checks:** ``curl -v`; load balancer target health; application access logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q26. How would you determine whether a timeout is caused by DNS lookup, TCP connect, TLS handshake, or server processing using curl timing?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** L4/L7, health checks, stickiness, X-Forwarded-For, timeouts Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Load balancer reports unhealthy targets. Check health path/status, target port, security rules, app bind address, and expected protocol.

**Commands / checks:** ``curl -v`; load balancer target health; application access logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q27. A certificate contains the right common name but the wrong SAN. Why can validation fail and how should the certificate be corrected?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** L4/L7, health checks, stickiness, X-Forwarded-For, timeouts Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Load balancer reports unhealthy targets. Check health path/status, target port, security rules, app bind address, and expected protocol.

**Commands / checks:** ``curl -v`; load balancer target health; application access logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q28. A service is reachable by IP but not hostname from a private subnet. What private DNS zone and resolver association checks matter?

**Difficulty:** Advanced

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** L4/L7, health checks, stickiness, X-Forwarded-For, timeouts Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Load balancer reports unhealthy targets. Check health path/status, target port, security rules, app bind address, and expected protocol.

**Commands / checks:** ``curl -v`; load balancer target health; application access logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q29. Retries at multiple proxy layers multiply traffic during an outage. How would you set retry budgets, backoff, and idempotence rules?

**Difficulty:** Intermediate

**Interview-ready answer:** Correlate proxy timing and upstream logs; compare connect/read/idle timeouts, connection reuse, target health, and graceful draining across each hop. Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** L4/L7, health checks, stickiness, X-Forwarded-For, timeouts Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Correlate proxy timing and upstream logs; compare connect/read/idle timeouts, connection reuse, target health, and graceful draining across each hop.

**Production scenario:** Load balancer reports unhealthy targets. Check health path/status, target port, security rules, app bind address, and expected protocol.

**Commands / checks:** ``curl -v`; load balancer target health; application access logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q30. A CDN or proxy caches an outdated response after deployment. What cache keys, TTLs, invalidation, and origin headers would you inspect?

**Difficulty:** Advanced

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** L4/L7, health checks, stickiness, X-Forwarded-For, timeouts Understand where TLS terminates and how health checks, target ports, connection draining, timeouts, and proxy headers affect requests. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Load balancer reports unhealthy targets. Check health path/status, target port, security rules, app bind address, and expected protocol.

**Commands / checks:** ``curl -v`; load balancer target health; application access logs`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q31. A backend closes idle connections and the load balancer returns intermittent 502s. How do idle timeout settings interact across layers?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, CA chain, SAN, expiry, SNI, mTLS, cipher compatibility Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Certificate works in browser but fails in a container. Check container CA bundle, hostname/SAN, missing intermediate chain, proxy, and clock skew.

**Commands / checks:** ``openssl s_client -connect host:443 -servername host -showcerts`; `curl -Iv``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q32. An IPv6 client fails while IPv4 works. What DNS AAAA, routing, security, and listener checks would you make?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, CA chain, SAN, expiry, SNI, mTLS, cipher compatibility Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Certificate works in browser but fails in a container. Check container CA bundle, hostname/SAN, missing intermediate chain, proxy, and clock skew.

**Commands / checks:** ``openssl s_client -connect host:443 -servername host -showcerts`; `curl -Iv``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q33. An application sees a certificate from an unexpected issuer. How would you inspect SNI, proxy interception, trust store, and certificate chain?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, CA chain, SAN, expiry, SNI, mTLS, cipher compatibility Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Certificate works in browser but fails in a container. Check container CA bundle, hostname/SAN, missing intermediate chain, proxy, and clock skew.

**Commands / checks:** ``openssl s_client -connect host:443 -servername host -showcerts`; `curl -Iv``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q34. A DNS record change appears to work in one region but not another. How do resolver geography and split-horizon DNS affect results?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, CA chain, SAN, expiry, SNI, mTLS, cipher compatibility Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Certificate works in browser but fails in a container. Check container CA bundle, hostname/SAN, missing intermediate chain, proxy, and clock skew.

**Commands / checks:** ``openssl s_client -connect host:443 -servername host -showcerts`; `curl -Iv``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q35. An API's TCP port is open but requests fail with 401. Why is this not a network reachability failure and what should be checked next?

**Difficulty:** Advanced

**Interview-ready answer:** Check the process's listening address and port, local socket state, host and cloud firewalls, route, and return path. Connection refused differs from a silent timeout. Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, CA chain, SAN, expiry, SNI, mTLS, cipher compatibility Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Check the process's listening address and port, local socket state, host and cloud firewalls, route, and return path. Connection refused differs from a silent timeout.

**Production scenario:** Certificate works in browser but fails in a container. Check container CA bundle, hostname/SAN, missing intermediate chain, proxy, and clock skew.

**Commands / checks:** ``openssl s_client -connect host:443 -servername host -showcerts`; `curl -Iv``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q36. A reverse proxy reports upstream connection refused. What does that indicate compared with connection timeout or HTTP 503?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, CA chain, SAN, expiry, SNI, mTLS, cipher compatibility Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Certificate works in browser but fails in a container. Check container CA bundle, hostname/SAN, missing intermediate chain, proxy, and clock skew.

**Commands / checks:** ``openssl s_client -connect host:443 -servername host -showcerts`; `curl -Iv``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q37. A load balancer drains a target but long-running requests fail. What connection draining and application shutdown behavior should be tested?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, CA chain, SAN, expiry, SNI, mTLS, cipher compatibility Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Certificate works in browser but fails in a container. Check container CA bundle, hostname/SAN, missing intermediate chain, proxy, and clock skew.

**Commands / checks:** ``openssl s_client -connect host:443 -servername host -showcerts`; `curl -Iv``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q38. How do you use `tcpdump` safely to investigate a production network issue while minimizing sensitive-data capture?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, CA chain, SAN, expiry, SNI, mTLS, cipher compatibility Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Certificate works in browser but fails in a container. Check container CA bundle, hostname/SAN, missing intermediate chain, proxy, and clock skew.

**Commands / checks:** ``openssl s_client -connect host:443 -servername host -showcerts`; `curl -Iv``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q39. A certificate renewal automation reports success but expiry alerts continue. How do you verify the certificate actually served on each endpoint?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, CA chain, SAN, expiry, SNI, mTLS, cipher compatibility Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Certificate works in browser but fails in a container. Check container CA bundle, hostname/SAN, missing intermediate chain, proxy, and clock skew.

**Commands / checks:** ``openssl s_client -connect host:443 -servername host -showcerts`; `curl -Iv``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q40. A private service should be reachable from one VPC but not another. What routing, peering/TGW, DNS, and security boundaries would you configure?

**Difficulty:** Advanced

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Handshake, CA chain, SAN, expiry, SNI, mTLS, cipher compatibility Validate certificate chain, SAN hostname, expiry, SNI, trust store, protocol compatibility, and key rotation. mTLS additionally validates client identity. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Certificate works in browser but fails in a container. Check container CA bundle, hostname/SAN, missing intermediate chain, proxy, and clock skew.

**Commands / checks:** ``openssl s_client -connect host:443 -servername host -showcerts`; `curl -Iv``

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q41. An API returns 413 for large requests. Which proxy/body-size limits and application limits would you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** HTTP status semantics, retries, keep-alive, CORS, idempotence Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Retries increase an outage. Use exponential backoff with jitter, retry budgets, timeouts, and circuit breakers; avoid retrying non-idempotent operations blindly.

**Commands / checks:** ``curl -sS -D - -o /dev/null -w '%{http_code} %{time_total}\n' URL`; packet capture`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q42. Clients see intermittent NXDOMAIN after a service rename. How do negative caching, rollout overlap, and stale clients matter?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** HTTP status semantics, retries, keep-alive, CORS, idempotence Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Retries increase an outage. Use exponential backoff with jitter, retry budgets, timeouts, and circuit breakers; avoid retrying non-idempotent operations blindly.

**Commands / checks:** ``curl -sS -D - -o /dev/null -w '%{http_code} %{time_total}\n' URL`; packet capture`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q43. A TLS certificate chain omits an intermediate CA. Why do some clients still work, and what is the correct fix?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** HTTP status semantics, retries, keep-alive, CORS, idempotence Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Retries increase an outage. Use exponential backoff with jitter, retry budgets, timeouts, and circuit breakers; avoid retrying non-idempotent operations blindly.

**Commands / checks:** ``curl -sS -D - -o /dev/null -w '%{http_code} %{time_total}\n' URL`; packet capture`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q44. A network path has asymmetric routing through two firewalls. Why can established connections fail despite outbound rules appearing open?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** HTTP status semantics, retries, keep-alive, CORS, idempotence Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Retries increase an outage. Use exponential backoff with jitter, retry budgets, timeouts, and circuit breakers; avoid retrying non-idempotent operations blindly.

**Commands / checks:** ``curl -sS -D - -o /dev/null -w '%{http_code} %{time_total}\n' URL`; packet capture`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q45. A load balancer distributes traffic unevenly. How do keep-alive, stickiness, target health, connection counts, and request duration affect distribution?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** HTTP status semantics, retries, keep-alive, CORS, idempotence Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Retries increase an outage. Use exponential backoff with jitter, retry budgets, timeouts, and circuit breakers; avoid retrying non-idempotent operations blindly.

**Commands / checks:** ``curl -sS -D - -o /dev/null -w '%{http_code} %{time_total}\n' URL`; packet capture`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q46. How would you build a certificate-expiry monitoring check that validates the certificate actually served, not merely the file on disk?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock. Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** HTTP status semantics, retries, keep-alive, CORS, idempotence Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect the certificate actually served with SNI, SAN/hostname, validity, chain, trust store, EKU, protocol/cipher compatibility, and system clock.

**Production scenario:** Retries increase an outage. Use exponential backoff with jitter, retry budgets, timeouts, and circuit breakers; avoid retrying non-idempotent operations blindly.

**Commands / checks:** ``curl -sS -D - -o /dev/null -w '%{http_code} %{time_total}\n' URL`; packet capture`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q47. A service is accessible through a public hostname from inside the VPC but hairpins through a public endpoint. What private DNS design could improve this?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** HTTP status semantics, retries, keep-alive, CORS, idempotence Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Retries increase an outage. Use exponential backoff with jitter, retry budgets, timeouts, and circuit breakers; avoid retrying non-idempotent operations blindly.

**Commands / checks:** ``curl -sS -D - -o /dev/null -w '%{http_code} %{time_total}\n' URL`; packet capture`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q48. A proxy returns 502 after an upstream certificate rotation. What trust bundle, SNI, and connection reuse checks are needed?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout. Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** HTTP status semantics, retries, keep-alive, CORS, idempotence Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect listener/rule matching, target health reason, backend port/protocol, security rules, connection draining, and application logs. Distinguish no healthy target from upstream refusal or timeout.

**Production scenario:** Retries increase an outage. Use exponential backoff with jitter, retry budgets, timeouts, and circuit breakers; avoid retrying non-idempotent operations blindly.

**Commands / checks:** ``curl -sS -D - -o /dev/null -w '%{http_code} %{time_total}\n' URL`; packet capture`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q49. How would you prove whether a 5-second delay comes from DNS, connect, TLS, or first-byte latency?

**Difficulty:** Intermediate

**Interview-ready answer:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network. Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** HTTP status semantics, retries, keep-alive, CORS, idempotence Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Compare authoritative records, recursive resolver answers, TTLs, negative caching, split-horizon zones, and client caches from the affected network.

**Production scenario:** Retries increase an outage. Use exponential backoff with jitter, retry budgets, timeouts, and circuit breakers; avoid retrying non-idempotent operations blindly.

**Commands / checks:** ``curl -sS -D - -o /dev/null -w '%{http_code} %{time_total}\n' URL`; packet capture`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q50. Describe a production incident where you isolate a networking issue layer by layer and validate that the fix did not weaken security.

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** HTTP status semantics, retries, keep-alive, CORS, idempotence Use status codes, response headers, timing breakdown, and server logs to isolate failures. Retry only transient failures and respect idempotence and backoff. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Retries increase an outage. Use exponential backoff with jitter, retry budgets, timeouts, and circuit breakers; avoid retrying non-idempotent operations blindly.

**Commands / checks:** ``curl -sS -D - -o /dev/null -w '%{http_code} %{time_total}\n' URL`; packet capture`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

## Official references

- [RFC 8446: TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446)
- [Let's Encrypt documentation](https://letsencrypt.org/docs/)
- [Cloudflare Learning Center](https://www.cloudflare.com/learning/)

## Version and safety note

This guide is dated 18 October 2026. Release status, feature maturity, cloud-region availability, plugins, and supported versions change. Check the linked official documentation against the exact versions and architecture you use before applying any production recommendation.
