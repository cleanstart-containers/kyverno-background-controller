# Kyverno Background Controller Container

This container package ships the custom `cleanstart/kyverno-background-controller:latest-dev` image
and a ready-to-run Kubernetes sample that demonstrates how to run Kyverno’s Background Controller
with correct CRDs, RBAC, health probes, and metrics.

## Contents

- `kubernetes/deployment.yaml` – Namespace, RBAC, Service and Deployment for the controller.
- `kubernetes/README.md` – Detailed, step-by-step walkthrough for installing CRDs, deploying,
  verifying readiness and accessing metrics.

## Prerequisites

- Running Kubernetes cluster (Kind, minikube, k3s, or cloud)
- `kubectl` configured for the target cluster
- Network access to the cluster

## Quick Start

1. Install Kyverno CRDs (v1.13, server-side apply) – ensures legacy CRDs exist and bypasses annotation size limits.
2. Remove any existing Kyverno background controller – avoids immutable selector conflicts.
3. Deploy the sample – apply `kubernetes/deployment.yaml` to install Namespace, RBAC, Service, and Deployment.
4. Verify – ensure the pod in namespace `kyverno` becomes Ready and logs show controllers started and caches synced.
5. Access metrics (optional) – port-forward the service on port 8080 and curl `/metrics`.

The `kubernetes/README.md` file documents the exact commands and expected outputs for each step.

## Project Highlights

- **Purpose-built image** – ships `cleanstart/kyverno-background-controller:latest-dev` with the controller binary.
- **Ready-to-run sample** – one manifest sets up namespace, RBAC, Service, Deployment, and probes aligned to 8080.
+- **CRD compatibility** – instructions pin to Kyverno v1.13 CRDs and use server-side apply to avoid size limits.
- **Operator-friendly** – includes verification steps, metrics access, and focused troubleshooting.

## What Gets Deployed

- Namespace `kyverno` and least-privilege RBAC for the background controller.
- Deployment running the Kyverno background controller with readiness/liveness on port 8080.
- ClusterIP Service exposing ports 8080 (metrics/health) and 9443 (HTTPS, optional).

## Cleanup

Remove the deployed resources by deleting the sample manifest:
```bash
kubectl delete -f kubernetes/deployment.yaml
```
If you installed Kyverno only for this test, you may also remove the `kyverno` namespace as needed.

## Troubleshooting

- **CRDs missing** – re-apply Kyverno v1.13 install with server-side apply and verify `policies.kyverno.io` and `clusterpolicies.kyverno.io` exist.
- **Immutable selector on apply** – delete the existing `kyverno-background-controller` Deployment in the `kyverno` namespace, then re-apply.
- **Probes failing / pod 0/1** – ensure readiness/liveness probes reference port 8080 (matches this sample).
- **kubectl top unavailable** – install metrics-server; Kind/minikube may require insecure TLS flags.

Detailed commands and outputs are provided in `kubernetes/README.md`.