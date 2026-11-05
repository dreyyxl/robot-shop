# Incident Postmortem: Robot Shop Payment Service Outage

## 1. Incident Summary

**Date:** November 5, 2026  
**Application:** Instana Robot Shop  
**Affected service:** Payment  
**Environment:** Local Kubernetes Kind cluster  
**Incident type:** Controlled outage simulation  
**Severity:** Simulated critical incident  
**Status:** Resolved

A controlled incident was performed to validate the observability stack's ability to detect and recover from a Payment service outage.

The Payment deployment was deliberately scaled to zero replicas, making the service unavailable.

Prometheus successfully detected the failure and triggered the `RobotShopPaymentDown` alert.

The deployment was subsequently restored to one replica, and Prometheus confirmed recovery.

## 2. Root Cause

The outage was intentionally introduced using:

```bash
kubectl scale deployment/payment --replicas=0 -n robot-shop
```

This removed all running Payment replicas.

There was no unexpected application failure.

## 3. Detection

The outage was detected using a custom Prometheus alert rule:

- **Alert:** `RobotShopPaymentDown`
- **Severity:** Critical
- **Condition:** Payment scrape target unavailable or absent
- **Configured alert duration:** 1 minute

The alert successfully transitioned to **FIRING** during the simulated outage.

## 4. Recovery

The Payment deployment was restored using:

```bash
kubectl scale deployment/payment --replicas=1 -n robot-shop
```

Kubernetes subsequently confirmed:

```text
deployment "payment" successfully rolled out
```

Prometheus later showed the alert returning to **INACTIVE**.

## 5. Alertmanager Verification

Alertmanager was successfully deployed and became healthy.

Prometheus confirmed an active Alertmanager endpoint on port `9093`.

Notification metrics showed:

- `prometheus_notifications_sent_total`: 491
- Notification errors during the checked 10-minute window: 0

These metrics confirm successful notification communication generally, but delivery of the specific `RobotShopPaymentDown` alert was not individually verified.

External notification delivery was not tested.

## 6. Lessons Learned

- Prometheus can detect service availability failures automatically.
- Kubernetes deployments can be restored by scaling replicas.
- Alert rules should be tested against real failure scenarios.
- Alertmanager startup delays can occur during container image downloads.
- Alert delivery should be verified separately from alert evaluation.

## 7. Recommended Improvements

- Configure an external notification receiver.
- Verify the Payment alert directly in Alertmanager during a future test.
- Add application latency and error-rate alerts.
- Establish measurable service-level indicators and objectives.
- Automate incident response testing where practical.

## 8. Conclusion

The controlled incident successfully demonstrated Payment service outage detection, alert firing, Kubernetes recovery, and alert resolution.

The exercise validated the core Prometheus alerting workflow while identifying additional notification verification and monitoring improvements.