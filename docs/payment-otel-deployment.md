# Payment OpenTelemetry Deployment Guide

## Overview

The Robot Shop Payment service has been instrumented with OpenTelemetry to export distributed traces to Jaeger.

Custom Docker image: `robotshop/rs-payment:otel-v2`

This image is built locally and loaded into the Kind Kubernetes cluster.

## 1. Build the Docker Image

From the Robot Shop repository root, run:

```bash
docker build -t robotshop/rs-payment:otel-v2 ./payment
```

This builds the Payment application with its Python and OpenTelemetry dependencies.

## 2. Load the Image into Kind

```bash
kind load docker-image robotshop/rs-payment:otel-v2 --name voting-cluster
```

This makes the locally built image available to the Kind cluster.

Repeat this step if the cluster is recreated.

## 3. Configure Helm

The Payment image tag is defined in `K8s/helm/values.yaml`:

```yaml
payment:
  imageTag: otel-v2
```

The Payment deployment template uses this value to select the custom image.

## 4. Deploy the Payment Image

For an existing Robot Shop deployment:

```bash
kubectl set image deployment/payment payment=robotshop/rs-payment:otel-v2 -n robot-shop
```

Verify the rollout:

```bash
kubectl rollout status deployment/payment -n robot-shop
```

Note: The Helm template also selects `otel-v2`, but the image must be loaded into Kind before deploying or upgrading the application through Helm.

## 5. Verify the Running Image

```bash
kubectl get deployment payment -n robot-shop -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

Expected output:

```text
robotshop/rs-payment:otel-v2
```

## 6. Verify Tracing

Forward the Jaeger UI:

```bash
kubectl port-forward -n monitoring svc/jaeger 16686:16686
```

Open `http://localhost:16686`.

Generate Payment HTTP requests and search for the `payment` service in Jaeger.

The current instrumentation captures Payment HTTP requests and supported outbound HTTP calls. Other Robot Shop services require their own instrumentation for complete cross-service traces.

## Important Limitations

- The custom image is currently local and is not published to a container registry.
- The Kind cluster must have the image loaded before deployment.
- Python dependencies are not pinned to exact versions.
- Jaeger uses in-memory storage, so traces are not retained after its storage is restarted.
- This guide describes the local development environment, not a production deployment.
