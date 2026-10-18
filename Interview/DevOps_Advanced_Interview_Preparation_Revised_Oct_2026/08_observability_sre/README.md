# Prometheus, Grafana, OpenTelemetry & SRE — 50 Advanced Interview Questions

**Audience:** Mid-level/L2 DevOps Engineer (3–5 years).
**Reference date:** 18 October 2026.

**Focus:** Prometheus, Grafana, OpenTelemetry & SRE production operations, troubleshooting, security, and reliability

## How to practise
- First answer the **interview-ready answer** aloud in 60–90 seconds.
- Then explain the technical reasoning, trade-offs, and evidence you would collect.
- Treat commands as diagnostic starting points; replace placeholders and validate permissions.
- Do not run destructive or mutating commands in production without an approved change plan.
- Use the follow-up to test whether you understand the failure mechanism rather than memorizing a script.

## Questions

### Q1. Prometheus memory usage grows sharply after a new label was added. How do you identify and reduce cardinality?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions. Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Counters, gauges, histograms, rate, aggregation, labels Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions.

**Production scenario:** Prometheus memory usage spikes after adding labels. Inspect series count and labels such as user IDs or request IDs; replace unbounded labels with bounded dimensions.

**Commands / checks:** ``rate(http_requests_total[5m])`; `histogram_quantile`; Prometheus query UI`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q2. p95 latency doubles but average latency is unchanged. How do you use histogram metrics and traces to find the affected requests?

**Difficulty:** Intermediate

**Interview-ready answer:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions. Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Counters, gauges, histograms, rate, aggregation, labels Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions.

**Production scenario:** Prometheus memory usage spikes after adding labels. Inspect series count and labels such as user IDs or request IDs; replace unbounded labels with bounded dimensions.

**Commands / checks:** ``rate(http_requests_total[5m])`; `histogram_quantile`; Prometheus query UI`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q3. Alertmanager pages on every replica restart. How would you redesign alerts around user impact and actionable symptoms?

**Difficulty:** Advanced

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Counters, gauges, histograms, rate, aggregation, labels Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** Prometheus memory usage spikes after adding labels. Inspect series count and labels such as user IDs or request IDs; replace unbounded labels with bounded dimensions.

**Commands / checks:** ``rate(http_requests_total[5m])`; `histogram_quantile`; Prometheus query UI`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q4. Prometheus shows gaps in a target's metrics. How do you distinguish scrape failure, target down, relabeling, and ingestion problems?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions. Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Counters, gauges, histograms, rate, aggregation, labels Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions.

**Production scenario:** Prometheus memory usage spikes after adding labels. Inspect series count and labels such as user IDs or request IDs; replace unbounded labels with bounded dimensions.

**Commands / checks:** ``rate(http_requests_total[5m])`; `histogram_quantile`; Prometheus query UI`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q5. An OpenTelemetry Collector queue fills during a backend outage. What retry, queue, memory limiter, and persistence settings would you inspect?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect receiver and exporter pipeline wiring, processor order, memory limiter, batch size, queue depth, retry behavior, and backend endpoint/authentication. Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Counters, gauges, histograms, rate, aggregation, labels Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect receiver and exporter pipeline wiring, processor order, memory limiter, batch size, queue depth, retry behavior, and backend endpoint/authentication.

**Production scenario:** Prometheus memory usage spikes after adding labels. Inspect series count and labels such as user IDs or request IDs; replace unbounded labels with bounded dimensions.

**Commands / checks:** ``rate(http_requests_total[5m])`; `histogram_quantile`; Prometheus query UI`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q6. Logs and traces cannot be correlated across two services. What trace-context propagation and resource-attribute checks matter?

**Difficulty:** Intermediate

**Interview-ready answer:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions. Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Counters, gauges, histograms, rate, aggregation, labels Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions.

**Production scenario:** Prometheus memory usage spikes after adding labels. Inspect series count and labels such as user IDs or request IDs; replace unbounded labels with bounded dimensions.

**Commands / checks:** ``rate(http_requests_total[5m])`; `histogram_quantile`; Prometheus query UI`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q7. An SLO burn-rate alert fires during a brief low-traffic spike. How do you tune windows and minimum traffic safeguards?

**Difficulty:** Intermediate

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Counters, gauges, histograms, rate, aggregation, labels Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** Prometheus memory usage spikes after adding labels. Inspect series count and labels such as user IDs or request IDs; replace unbounded labels with bounded dimensions.

**Commands / checks:** ``rate(http_requests_total[5m])`; `histogram_quantile`; Prometheus query UI`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q8. Grafana dashboards show different values for the same metric. What query range, rate window, labels, and aggregation differences could explain it?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions. Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Counters, gauges, histograms, rate, aggregation, labels Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions.

**Production scenario:** Prometheus memory usage spikes after adding labels. Inspect series count and labels such as user IDs or request IDs; replace unbounded labels with bounded dimensions.

**Commands / checks:** ``rate(http_requests_total[5m])`; `histogram_quantile`; Prometheus query UI`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q9. An exporter is healthy but Prometheus reports target down. How do you inspect scrape path, TLS/auth, network policy, and service discovery?

**Difficulty:** Intermediate

**Interview-ready answer:** Check target discovery, scrape path, auth/TLS, relabeling, network access, sample ingestion, and backend capacity. Monitor the telemetry pipeline itself. Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Counters, gauges, histograms, rate, aggregation, labels Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Check target discovery, scrape path, auth/TLS, relabeling, network access, sample ingestion, and backend capacity. Monitor the telemetry pipeline itself.

**Production scenario:** Prometheus memory usage spikes after adding labels. Inspect series count and labels such as user IDs or request IDs; replace unbounded labels with bounded dimensions.

**Commands / checks:** ``rate(http_requests_total[5m])`; `histogram_quantile`; Prometheus query UI`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q10. How would you build a service dashboard that helps an on-call engineer answer what broke, who is affected, and what changed?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect query cost, label cardinality, time range, refresh interval, repeated panels, and backend load. Design dashboards around incident questions rather than vanity graphs. Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Counters, gauges, histograms, rate, aggregation, labels Choose metric types correctly, use `rate()` for counters, aggregate intentionally, and control label cardinality. Histogram buckets support percentile estimates; do not average precomputed quantiles blindly. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect query cost, label cardinality, time range, refresh interval, repeated panels, and backend load. Design dashboards around incident questions rather than vanity graphs.

**Production scenario:** Prometheus memory usage spikes after adding labels. Inspect series count and labels such as user IDs or request IDs; replace unbounded labels with bounded dimensions.

**Commands / checks:** ``rate(http_requests_total[5m])`; `histogram_quantile`; Prometheus query UI`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q11. An alert says CPU is high, but customers report timeouts and CPU is normal. What user-centric signals do you inspect next?

**Difficulty:** Intermediate

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** SLIs, SLOs, error budgets, burn-rate alerts, alert quality Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** On-call receives hundreds of alerts. Group related alerts, remove non-actionable pages, set inhibition, and prioritize symptoms over every internal component warning.

**Commands / checks:** `PromQL rules; Alertmanager routing; runbook links; SLO/error-budget dashboards`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q12. A PromQL query returns inflated request rates after adding a replica. How can label aggregation and counter resets cause mistakes?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions. Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** SLIs, SLOs, error budgets, burn-rate alerts, alert quality Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions.

**Production scenario:** On-call receives hundreds of alerts. Group related alerts, remove non-actionable pages, set inhibition, and prioritize symptoms over every internal component warning.

**Commands / checks:** `PromQL rules; Alertmanager routing; runbook links; SLO/error-budget dashboards`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q13. An OpenTelemetry Collector drops spans under peak load. How would you inspect batch processors, memory limits, sampling, and exporter queues?

**Difficulty:** Advanced

**Interview-ready answer:** Check target discovery, scrape path, auth/TLS, relabeling, network access, sample ingestion, and backend capacity. Monitor the telemetry pipeline itself. Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** SLIs, SLOs, error budgets, burn-rate alerts, alert quality Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Check target discovery, scrape path, auth/TLS, relabeling, network access, sample ingestion, and backend capacity. Monitor the telemetry pipeline itself.

**Production scenario:** On-call receives hundreds of alerts. Group related alerts, remove non-actionable pages, set inhibition, and prioritize symptoms over every internal component warning.

**Commands / checks:** `PromQL rules; Alertmanager routing; runbook links; SLO/error-budget dashboards`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q14. Logs contain trace IDs but the tracing backend has no matching spans. What sampling, context propagation, retention, and clock issues could explain this?

**Difficulty:** Intermediate

**Interview-ready answer:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions. Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** SLIs, SLOs, error budgets, burn-rate alerts, alert quality Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions.

**Production scenario:** On-call receives hundreds of alerts. Group related alerts, remove non-actionable pages, set inhibition, and prioritize symptoms over every internal component warning.

**Commands / checks:** `PromQL rules; Alertmanager routing; runbook links; SLO/error-budget dashboards`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q15. A metric label contains a customer ID and series count explodes. How do you reduce cardinality without losing operational value?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions. Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** SLIs, SLOs, error budgets, burn-rate alerts, alert quality Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions.

**Production scenario:** On-call receives hundreds of alerts. Group related alerts, remove non-actionable pages, set inhibition, and prioritize symptoms over every internal component warning.

**Commands / checks:** `PromQL rules; Alertmanager routing; runbook links; SLO/error-budget dashboards`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q16. An alert fires repeatedly but no one can take action from it. How do you decide whether to tune, ticket, or remove the alert?

**Difficulty:** Intermediate

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** SLIs, SLOs, error budgets, burn-rate alerts, alert quality Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** On-call receives hundreds of alerts. Group related alerts, remove non-actionable pages, set inhibition, and prioritize symptoms over every internal component warning.

**Commands / checks:** `PromQL rules; Alertmanager routing; runbook links; SLO/error-budget dashboards`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q17. An SLO is missed even though all infrastructure dashboards are green. What could infrastructure-only monitoring be missing?

**Difficulty:** Intermediate

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** SLIs, SLOs, error budgets, burn-rate alerts, alert quality Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** On-call receives hundreds of alerts. Group related alerts, remove non-actionable pages, set inhibition, and prioritize symptoms over every internal component warning.

**Commands / checks:** `PromQL rules; Alertmanager routing; runbook links; SLO/error-budget dashboards`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q18. How do you design burn-rate alerts for a service with a 99.9% availability SLO?

**Difficulty:** Advanced

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** SLIs, SLOs, error budgets, burn-rate alerts, alert quality Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** On-call receives hundreds of alerts. Group related alerts, remove non-actionable pages, set inhibition, and prioritize symptoms over every internal component warning.

**Commands / checks:** `PromQL rules; Alertmanager routing; runbook links; SLO/error-budget dashboards`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q19. An exporter endpoint becomes slow and causes Prometheus scrape timeouts. How do you prevent monitoring itself from overloading the target?

**Difficulty:** Intermediate

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** SLIs, SLOs, error budgets, burn-rate alerts, alert quality Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** On-call receives hundreds of alerts. Group related alerts, remove non-actionable pages, set inhibition, and prioritize symptoms over every internal component warning.

**Commands / checks:** `PromQL rules; Alertmanager routing; runbook links; SLO/error-budget dashboards`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q20. A Grafana dashboard becomes slow after adding many panels. How do you inspect query cost, refresh interval, cardinality, and backend load?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions. Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** SLIs, SLOs, error budgets, burn-rate alerts, alert quality Alert on user-impacting symptoms and actionable conditions. Use SLOs and multi-window burn-rate alerts to balance fast detection against noise. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions.

**Production scenario:** On-call receives hundreds of alerts. Group related alerts, remove non-actionable pages, set inhibition, and prioritize symptoms over every internal component warning.

**Commands / checks:** `PromQL rules; Alertmanager routing; runbook links; SLO/error-budget dashboards`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q21. An incident lasts 40 minutes but MTTR reports only 10 minutes. What event definitions and timestamps should be clarified?

**Difficulty:** Intermediate

**Interview-ready answer:** Define incident timestamps consistently, record mitigation and recovery separately, and turn postmortem actions into owned, time-bound, verifiable work. Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Correlation IDs, trace context, sampling, structured logs, privacy Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Define incident timestamps consistently, record mitigation and recovery separately, and turn postmortem actions into owned, time-bound, verifiable work.

**Production scenario:** Traces stop at a service boundary. Check context propagation, instrumentation, collector receivers/exporters, sampling, and network access.

**Commands / checks:** `OpenTelemetry SDK; OTLP endpoints; trace IDs in logs; backend trace search`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q22. A trace shows a long database span but the database metrics look normal. What sampling, query, pool, and instrumentation evidence would you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions. Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Correlation IDs, trace context, sampling, structured logs, privacy Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions.

**Production scenario:** Traces stop at a service boundary. Check context propagation, instrumentation, collector receivers/exporters, sampling, and network access.

**Commands / checks:** `OpenTelemetry SDK; OTLP endpoints; trace IDs in logs; backend trace search`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q23. Prometheus restarts and loses recent data. What retention, persistent storage, WAL, and recovery design would you review?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Correlation IDs, trace context, sampling, structured logs, privacy Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Traces stop at a service boundary. Check context propagation, instrumentation, collector receivers/exporters, sampling, and network access.

**Commands / checks:** `OpenTelemetry SDK; OTLP endpoints; trace IDs in logs; backend trace search`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q24. Alertmanager sends duplicate pages to two teams. How do grouping, routing, inhibition, and receiver configuration work together?

**Difficulty:** Intermediate

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Correlation IDs, trace context, sampling, structured logs, privacy Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** Traces stop at a service boundary. Check context propagation, instrumentation, collector receivers/exporters, sampling, and network access.

**Commands / checks:** `OpenTelemetry SDK; OTLP endpoints; trace IDs in logs; backend trace search`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q25. A collector receives OTLP but exports nothing. What receiver pipeline, processor, exporter, endpoint, and authentication checks are needed?

**Difficulty:** Advanced

**Interview-ready answer:** Check target discovery, scrape path, auth/TLS, relabeling, network access, sample ingestion, and backend capacity. Monitor the telemetry pipeline itself. Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Correlation IDs, trace context, sampling, structured logs, privacy Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Check target discovery, scrape path, auth/TLS, relabeling, network access, sample ingestion, and backend capacity. Monitor the telemetry pipeline itself.

**Production scenario:** Traces stop at a service boundary. Check context propagation, instrumentation, collector receivers/exporters, sampling, and network access.

**Commands / checks:** `OpenTelemetry SDK; OTLP endpoints; trace IDs in logs; backend trace search`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q26. How would you sample traces to control cost while retaining enough data for rare errors and tail latency?

**Difficulty:** Intermediate

**Interview-ready answer:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions. Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Correlation IDs, trace context, sampling, structured logs, privacy Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions.

**Production scenario:** Traces stop at a service boundary. Check context propagation, instrumentation, collector receivers/exporters, sampling, and network access.

**Commands / checks:** `OpenTelemetry SDK; OTLP endpoints; trace IDs in logs; backend trace search`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q27. A service's error budget is nearly exhausted. How should the team adjust release risk and prioritize reliability work?

**Difficulty:** Intermediate

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Correlation IDs, trace context, sampling, structured logs, privacy Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** Traces stop at a service boundary. Check context propagation, instrumentation, collector receivers/exporters, sampling, and network access.

**Commands / checks:** `OpenTelemetry SDK; OTLP endpoints; trace IDs in logs; backend trace search`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q28. An alert depends on a metric that disappears when the service is down. How do you design missing-series and absent-target detection?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions. Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Correlation IDs, trace context, sampling, structured logs, privacy Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions.

**Production scenario:** Traces stop at a service boundary. Check context propagation, instrumentation, collector receivers/exporters, sampling, and network access.

**Commands / checks:** `OpenTelemetry SDK; OTLP endpoints; trace IDs in logs; backend trace search`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q29. How do you prevent logs from leaking tokens or personal data while preserving useful incident context?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect receiver and exporter pipeline wiring, processor order, memory limiter, batch size, queue depth, retry behavior, and backend endpoint/authentication. Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Correlation IDs, trace context, sampling, structured logs, privacy Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Inspect receiver and exporter pipeline wiring, processor order, memory limiter, batch size, queue depth, retry behavior, and backend endpoint/authentication.

**Production scenario:** Traces stop at a service boundary. Check context propagation, instrumentation, collector receivers/exporters, sampling, and network access.

**Commands / checks:** `OpenTelemetry SDK; OTLP endpoints; trace IDs in logs; backend trace search`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q30. A postmortem action has been open for months. How would you make corrective actions measurable and accountable?

**Difficulty:** Advanced

**Interview-ready answer:** Define incident timestamps consistently, record mitigation and recovery separately, and turn postmortem actions into owned, time-bound, verifiable work. Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** Correlation IDs, trace context, sampling, structured logs, privacy Correlate telemetry using trace/span IDs and consistent resource attributes. Use structured logs, appropriate sampling, and redaction for sensitive fields. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Define incident timestamps consistently, record mitigation and recovery separately, and turn postmortem actions into owned, time-bound, verifiable work.

**Production scenario:** Traces stop at a service boundary. Check context propagation, instrumentation, collector receivers/exporters, sampling, and network access.

**Commands / checks:** `OpenTelemetry SDK; OTLP endpoints; trace IDs in logs; backend trace search`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q31. A new dashboard uses `avg()` over per-instance p99 values. Why may this misrepresent fleet-wide latency and what should be used instead?

**Difficulty:** Intermediate

**Interview-ready answer:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions. Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** OTel Collector receivers, processors, exporters, batching, queues Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions.

**Production scenario:** Telemetry drops during a backend outage. Check exporter queues, retries, memory limiter, persistent queue support, and backend capacity.

**Commands / checks:** `Collector health metrics; configuration validation; exporter error and queue metrics`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q32. Prometheus target discovery creates duplicate scrape targets. How would you inspect relabeling and service discovery labels?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions. Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** OTel Collector receivers, processors, exporters, batching, queues Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions.

**Production scenario:** Telemetry drops during a backend outage. Check exporter queues, retries, memory limiter, persistent queue support, and backend capacity.

**Commands / checks:** `Collector health metrics; configuration validation; exporter error and queue metrics`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q33. An OpenTelemetry pipeline sends the same logs twice after a config change. How do you investigate duplicate receivers, pipelines, and agents?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect receiver and exporter pipeline wiring, processor order, memory limiter, batch size, queue depth, retry behavior, and backend endpoint/authentication. Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** OTel Collector receivers, processors, exporters, batching, queues Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect receiver and exporter pipeline wiring, processor order, memory limiter, batch size, queue depth, retry behavior, and backend endpoint/authentication.

**Production scenario:** Telemetry drops during a backend outage. Check exporter queues, retries, memory limiter, persistent queue support, and backend capacity.

**Commands / checks:** `Collector health metrics; configuration validation; exporter error and queue metrics`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q34. An alert storm occurs during a regional outage. How would you design grouping and inhibition without hiding independent critical failures?

**Difficulty:** Intermediate

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** OTel Collector receivers, processors, exporters, batching, queues Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** Telemetry drops during a backend outage. Check exporter queues, retries, memory limiter, persistent queue support, and backend capacity.

**Commands / checks:** `Collector health metrics; configuration validation; exporter error and queue metrics`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q35. A service has a good availability SLO but poor latency. How should SLIs represent both dimensions?

**Difficulty:** Advanced

**Interview-ready answer:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions. Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** OTel Collector receivers, processors, exporters, batching, queues Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions.

**Production scenario:** Telemetry drops during a backend outage. Check exporter queues, retries, memory limiter, persistent queue support, and backend capacity.

**Commands / checks:** `Collector health metrics; configuration validation; exporter error and queue metrics`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q36. How would you detect a broken telemetry pipeline independently of the application it monitors?

**Difficulty:** Intermediate

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** OTel Collector receivers, processors, exporters, batching, queues Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Telemetry drops during a backend outage. Check exporter queues, retries, memory limiter, persistent queue support, and backend capacity.

**Commands / checks:** `Collector health metrics; configuration validation; exporter error and queue metrics`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q37. An exporter reports counters that reset on process restart. How should queries handle resets and rate windows?

**Difficulty:** Intermediate

**Interview-ready answer:** Check target discovery, scrape path, auth/TLS, relabeling, network access, sample ingestion, and backend capacity. Monitor the telemetry pipeline itself. Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** OTel Collector receivers, processors, exporters, batching, queues Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Check target discovery, scrape path, auth/TLS, relabeling, network access, sample ingestion, and backend capacity. Monitor the telemetry pipeline itself.

**Production scenario:** Telemetry drops during a backend outage. Check exporter queues, retries, memory limiter, persistent queue support, and backend capacity.

**Commands / checks:** `Collector health metrics; configuration validation; exporter error and queue metrics`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q38. A dashboard is accurate but unusable during an incident. How would you improve information hierarchy, annotations, and runbook links?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect query cost, label cardinality, time range, refresh interval, repeated panels, and backend load. Design dashboards around incident questions rather than vanity graphs. Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** OTel Collector receivers, processors, exporters, batching, queues Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Inspect query cost, label cardinality, time range, refresh interval, repeated panels, and backend load. Design dashboards around incident questions rather than vanity graphs.

**Production scenario:** Telemetry drops during a backend outage. Check exporter queues, retries, memory limiter, persistent queue support, and backend capacity.

**Commands / checks:** `Collector health metrics; configuration validation; exporter error and queue metrics`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q39. How do you define and measure toil for a platform team, and when is automation worth building?

**Difficulty:** Intermediate

**Interview-ready answer:** Define incident timestamps consistently, record mitigation and recovery separately, and turn postmortem actions into owned, time-bound, verifiable work. Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** OTel Collector receivers, processors, exporters, batching, queues Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Define incident timestamps consistently, record mitigation and recovery separately, and turn postmortem actions into owned, time-bound, verifiable work.

**Production scenario:** Telemetry drops during a backend outage. Check exporter queues, retries, memory limiter, persistent queue support, and backend capacity.

**Commands / checks:** `Collector health metrics; configuration validation; exporter error and queue metrics`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q40. A high-cardinality trace attribute contains raw URLs with IDs. How do you normalize attributes and protect privacy?

**Difficulty:** Advanced

**Interview-ready answer:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions. Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** OTel Collector receivers, processors, exporters, batching, queues Use the Collector to standardize telemetry pipelines and apply batching, filtering, enrichment, and export. Monitor its own queue, memory, and export failures. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Inspect series count and label values. Unbounded labels such as customer IDs, raw URLs, or request IDs can multiply time series; replace them with bounded dimensions.

**Production scenario:** Telemetry drops during a backend outage. Check exporter queues, retries, memory limiter, persistent queue support, and backend capacity.

**Commands / checks:** `Collector health metrics; configuration validation; exporter error and queue metrics`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q41. A deployment causes a latency regression but no error-rate increase. What release annotations and baseline comparisons help identify it?

**Difficulty:** Intermediate

**Interview-ready answer:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions. Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** MTTR, postmortems, capacity, toil, resilience testing Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions.

**Production scenario:** Incident recurs after a postmortem. Check whether actions have owners, deadlines, measurable outcomes, and verification.

**Commands / checks:** `Timeline, dashboards, deployment history, runbooks, error budget, postmortem actions`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q42. A collector's memory limiter drops data while the backend is healthy. What sizing, batch, queue, and traffic-spike factors do you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Inspect receiver and exporter pipeline wiring, processor order, memory limiter, batch size, queue depth, retry behavior, and backend endpoint/authentication. Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** MTTR, postmortems, capacity, toil, resilience testing Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Inspect receiver and exporter pipeline wiring, processor order, memory limiter, batch size, queue depth, retry behavior, and backend endpoint/authentication.

**Production scenario:** Incident recurs after a postmortem. Check whether actions have owners, deadlines, measurable outcomes, and verification.

**Commands / checks:** `Timeline, dashboards, deployment history, runbooks, error budget, postmortem actions`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q43. An SRE team wants 100% tracing for all traffic. How would you explain cost, overhead, sampling, and tail-based trade-offs?

**Difficulty:** Advanced

**Interview-ready answer:** Verify W3C trace-context propagation across service boundaries, SDK instrumentation, sampling policy, trace IDs in structured logs, and backend retention. Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** MTTR, postmortems, capacity, toil, resilience testing Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Verify W3C trace-context propagation across service boundaries, SDK instrumentation, sampling policy, trace IDs in structured logs, and backend retention.

**Production scenario:** Incident recurs after a postmortem. Check whether actions have owners, deadlines, measurable outcomes, and verification.

**Commands / checks:** `Timeline, dashboards, deployment history, runbooks, error budget, postmortem actions`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q44. A critical alert was acknowledged but not resolved because the runbook was outdated. How would you improve runbook quality and test it?

**Difficulty:** Intermediate

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** MTTR, postmortems, capacity, toil, resilience testing Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** Incident recurs after a postmortem. Check whether actions have owners, deadlines, measurable outcomes, and verification.

**Commands / checks:** `Timeline, dashboards, deployment history, runbooks, error budget, postmortem actions`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q45. A service has no agreed SLO. How would you choose user-visible SLIs and set a realistic target?

**Difficulty:** Advanced

**Interview-ready answer:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation. Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** MTTR, postmortems, capacity, toil, resilience testing Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Alert on actionable user-impacting symptoms, group related alerts, add inhibition carefully, and use SLO burn-rate windows. Avoid paging on every component fluctuation.

**Production scenario:** Incident recurs after a postmortem. Check whether actions have owners, deadlines, measurable outcomes, and verification.

**Commands / checks:** `Timeline, dashboards, deployment history, runbooks, error budget, postmortem actions`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

### Q46. How do you distinguish an incident symptom from a root cause in a blameless postmortem?

**Difficulty:** Intermediate

**Interview-ready answer:** Define incident timestamps consistently, record mitigation and recovery separately, and turn postmortem actions into owned, time-bound, verifiable work. Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** MTTR, postmortems, capacity, toil, resilience testing Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Begin with impact and scope, then collect evidence from the control plane and runtime, compare a known-good example, apply the smallest reversible mitigation, and verify recovery against an explicit success criterion. **Scenario-specific check:** Define incident timestamps consistently, record mitigation and recovery separately, and turn postmortem actions into owned, time-bound, verifiable work.

**Production scenario:** Incident recurs after a postmortem. Check whether actions have owners, deadlines, measurable outcomes, and verification.

**Commands / checks:** `Timeline, dashboards, deployment history, runbooks, error budget, postmortem actions`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which observation would falsify your leading hypothesis?

---

### Q47. Prometheus remote write falls behind. What queue, network, receiver capacity, and sample-rate signals do you inspect?

**Difficulty:** Intermediate

**Interview-ready answer:** Check target discovery, scrape path, auth/TLS, relabeling, network access, sample ingestion, and backend capacity. Monitor the telemetry pipeline itself. Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** MTTR, postmortems, capacity, toil, resilience testing Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Distinguish symptom from root cause by testing one hypothesis at a time. Preserve timestamps, correlate logs and metrics, and record what evidence would disprove the current hypothesis. **Scenario-specific check:** Check target discovery, scrape path, auth/TLS, relabeling, network access, sample ingestion, and backend capacity. Monitor the telemetry pipeline itself.

**Production scenario:** Incident recurs after a postmortem. Check whether actions have owners, deadlines, measurable outcomes, and verification.

**Commands / checks:** `Timeline, dashboards, deployment history, runbooks, error budget, postmortem actions`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What is the safest mitigation while the root cause is still uncertain?

---

### Q48. An application emits metrics with inconsistent service names across environments. How do resource attributes and naming conventions help?

**Difficulty:** Advanced

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** MTTR, postmortems, capacity, toil, resilience testing Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. For a production change, define a canary or limited blast radius, require review for high-risk actions, and retain a tested rollback or forward-recovery path. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Incident recurs after a postmortem. Check whether actions have owners, deadlines, measurable outcomes, and verification.

**Commands / checks:** `Timeline, dashboards, deployment history, runbooks, error budget, postmortem actions`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** How would you test the fix without increasing production blast radius?

---

### Q49. A synthetic check is green while real users fail. What differences in geography, auth, data, and request paths should be investigated?

**Difficulty:** Intermediate

**Interview-ready answer:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident. Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** MTTR, postmortems, capacity, toil, resilience testing Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Validate the result with both configuration evidence and runtime behavior; a successful command or green health check alone is not proof of end-to-end correctness. **Scenario-specific check:** Start by isolating the failure domain, gather evidence from configuration and runtime behavior, apply the smallest reversible mitigation, and verify the user-visible outcome before closing the incident.

**Production scenario:** Incident recurs after a postmortem. Check whether actions have owners, deadlines, measurable outcomes, and verification.

**Commands / checks:** `Timeline, dashboards, deployment history, runbooks, error budget, postmortem actions`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** Which metric or audit record would prove the issue is resolved?

---

### Q50. Describe an incident response where you use metrics, logs, traces, deployment events, and SLOs together.

**Difficulty:** Advanced

**Interview-ready answer:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions. Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. In an interview, connect it to availability, security, repeatability, or delivery speed rather than defining the tool alone.

**Technical explanation:** MTTR, postmortems, capacity, toil, resilience testing Define incident roles, mitigate impact first, preserve evidence, communicate clearly, and conduct blameless postmortems with tracked corrective actions. Start by establishing the desired state and blast radius, then validate assumptions with observable evidence. Prefer documented, repeatable procedures over one-off manual changes. Prevent recurrence with a measurable alert or policy, an owner for the corrective action, and a test that reproduces the original failure mode. **Scenario-specific check:** Use histogram buckets for aggregatable latency percentiles and distributed traces to identify which span dominates. Average latency can hide tail regressions.

**Production scenario:** Incident recurs after a postmortem. Check whether actions have owners, deadlines, measurable outcomes, and verification.

**Commands / checks:** `Timeline, dashboards, deployment history, runbooks, error budget, postmortem actions`

**Common mistake:** Changing several variables at once, relying on a single green status, or applying a destructive fix before confirming the failure domain.

**Follow-up:** What guardrail would prevent the same failure from recurring?

---

## Official references

- [Prometheus documentation](https://prometheus.io/docs/)
- [OpenTelemetry observability primer](https://opentelemetry.io/docs/concepts/observability-primer/)
- [Google SRE books](https://sre.google/books/)

## Version and safety note

This guide is dated 18 October 2026. Release status, feature maturity, cloud-region availability, plugins, and supported versions change. Check the linked official documentation against the exact versions and architecture you use before applying any production recommendation.
