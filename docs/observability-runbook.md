# Robot Shop — Observability SLO Policy and Incident Runbook

## 1. Purpose

This document defines proposed service-level objectives (SLOs), alert thresholds, troubleshooting procedures, and incident-response practices for the Robot Shop microservices application deployed on Kubernetes.

**Implementation status:** Prometheus and Grafana are deployed. Cart and Payment application metrics have been verified. The SLOs and alerts described below are proposed operational policies; they have not yet been fully implemented or validated.

## 2. System Overview

- **Platform:** Kubernetes using Kind
- **Deployment:** Helm
- **Application namespace:** `robot-shop`
- **Monitoring namespace:** `monitoring`
- **Monitoring stack:** Prometheus and Grafana
- **Focus services:** Web, Catalogue, Cart, and Payment

## 3. Proposed Service-Level Objectives

| Service | SLI | Proposed SLO |
|---|---|---|
| Web | Successful HTTP requests / total HTTP requests | 99.9% success over 30 days |
| Catalogue | Successful HTTP requests / total HTTP requests | 99.5% success over 30 days |
| Cart | Successful HTTP requests / total HTTP requests | 99.9% success over 30 days |
| Payment | Successful HTTP requests / total HTTP requests | 99.95% success over 30 days |
| Web | 95th percentile request latency | Under 500 ms |
| Catalogue | 95th percentile request latency | Under 400 ms |
| Cart | 95th percentile request latency | Under 500 ms |
| Payment | 95th percentile request latency | Under 800 ms |

These are project targets, not measured production performance.

The current instrumentation does not yet provide complete HTTP request and latency metrics for all four services.

## 4. Error-Budget Policy

For a 30-day period:

| Availability SLO | Allowed Unavailability |
|---|---|
| 99.9% | 43.2 minutes |
| 99.5% | 216 minutes |
| 99.95% | 21.6 minutes |

If a service exhausts its error budget, nonessential deployments should be paused until reliability is restored.

These figures assume a continuously operating service and represent time-equivalent error budgets.

## 5. Proposed Alert Rules

### Critical: Payment service unavailable

**Condition:** Payment's Prometheus `up` metric equals zero continuously for 2 minutes.

**Response:** Check the Payment Pod, Service, logs, and dependencies.

### Warning: High HTTP error rate

**Condition:** More than 5% of requests fail over a 5-minute window.

**Response:** Investigate recent deployments, application logs, and dependency health.

### Warning: High request latency

**Condition:** The service's 95th percentile request latency exceeds its proposed target for 5 minutes.

**Response:** Investigate resource utilization, dependency latency, and traffic volume.

### Critical: Pod restart loop

**Condition:** A focus-service Pod repeatedly restarts or enters `CrashLoopBackOff`.

**Response:** Inspect Pod events and logs, then determine whether rollback is needed.

**Note:** These rules are proposed. Slack delivery and full alert-rule deployment have not been configured or verified.

## 6. Troubleshooting Runbook

### Step 1 — Check Pod health

```bash
kubectl get pods -n robot-shop
```

Look for `Pending`, `Error`, `CrashLoopBackOff`, or unexpected restarts.

### Step 2 — Check service availability

```bash
kubectl get svc -n robot-shop
```

Confirm the affected service exists and exposes the expected port.

### Step 3 — Inspect application logs

For Payment:

```bash
kubectl logs -n robot-shop deployment/payment --tail=100
```

For Cart:

```bash
kubectl logs -n robot-shop deployment/cart --tail=100
```

### Step 4 — Inspect Kubernetes events

```bash
kubectl get events -n robot-shop --sort-by=.lastTimestamp
```

Check for image-pull failures, scheduling problems, or resource constraints.

### Step 5 — Check monitoring components

```bash
kubectl get pods -n monitoring
```

Verify Grafana, Prometheus, and Prometheus Operator are healthy.

### Step 6 — Investigate metrics

In Grafana Explore, query:

```promql
up{namespace="robot-shop"}
```

For Cart activity:

```promql
items_added
```

For Payment activity:

```promql
sold_count_total
```

A healthy scrape target reports `up = 1`. This confirms Prometheus can scrape the target, not necessarily that every application function is healthy.

### Step 7 — Recovery and verification

After identifying the cause, apply an appropriate fix or rollback.

Verify Pod readiness, service functionality, and relevant metrics before closing the incident.

## 7. Load-Test Evidence

The repository's existing Kubernetes load generator was deployed with three configured clients, a five-minute run duration, and error simulation enabled.

The load generator started successfully and was subsequently removed.

Metrics observed during the exercise:

| Service | Metric | Observed value |
|---|---|---|
| Cart | `items_added` | 18 |
| Payment | `sold_count_total` | 15.0 |

These values are cumulative counters. No baseline measurements were captured before the test, so the observed totals cannot be attributed entirely to this test.

The test did not establish HTTP error rates, latency percentiles, or an SLO compliance result.

## 8. Incident Drill and Blameless Postmortem Template

**Status:** Template only. No completed incident simulation or validated postmortem is claimed.

### Incident summary

- Incident date:
- Affected service:
- Severity:
- Start time:
- Detection time:
- Recovery time:

### Customer impact

Describe which user-facing functions were affected and how impact was measured.

### Timeline

Record the sequence of alerts, observations, investigation steps, mitigation actions, and recovery verification.

### Root cause

Document the technical and process factors that contributed to the incident, without assigning personal blame.

### What went well

Record effective detection, communication, troubleshooting, and recovery practices.

### What could improve

Identify gaps in instrumentation, alerts, procedures, and deployment safeguards.

### Follow-up actions

Assign owners and target dates for corrective actions.

## 9. Known Gaps

The following remain incomplete or unverified:

- Full HTTP request-rate, error-rate, and latency instrumentation for all four focus services
- Complete Grafana dashboard
- Fluent Bit and Loki centralized logging
- OpenTelemetry and Jaeger distributed tracing
- Deployed SLO alert rules and Slack notifications
- Completed incident drill and postmortem

This runbook documents the proposed operational approach and the evidence collected so far.