# Kyverno Background Controller - Complete Guide

## Overview
Deploy Kyverno's Background Controller in a Kubernetes cluster using the `cleanstart/kyverno-background-controller:latest-dev` image. This controller processes background policy evaluations and reporting.

---

## Prerequisites

- Kubernetes cluster (Kind, minikube, k3s, or cloud)
- kubectl installed and configured

---

## Step-by-Step Deployment

### Step 1: Verify Your Cluster
Confirm kubectl connectivity and that nodes are Ready.

```bash
kubectl cluster-info
kubectl get nodes
```

---

### Step 2: Install Kyverno CRDs (v1.13 — server-side apply)

```bash
curl -L https://github.com/kyverno/kyverno/releases/download/v1.13.0/install.yaml -o install.yaml
kubectl apply --server-side --force-conflicts -f install.yaml
```
This command will create install.yaml and apply necessary configuration.

Kyverno v1.13 provides the legacy CRDs this controller expects.

Server-side apply bypasses large-annotation limits that can break normal apply.

### Verify required CRDs

```bash
kubectl get crd policies.kyverno.io
kubectl get crd clusterpolicies.kyverno.io
```

---

### Step 3: Deploy Background Controller

Remove Kyverno's own background controller first to avoid "spec.selector is immutable" conflicts.
Then apply this repo’s deployment which uses correct probes and RBAC.

```bash
kubectl -n kyverno delete deployment kyverno-background-controller --ignore-not-found
kubectl apply -f deployment.yaml
```

Expected output (example):
```
namespace/kyverno created|configured
serviceaccount/kyverno-background-controller created|configured
clusterrole.rbac.authorization.k8s.io/kyverno-background-controller created|configured
clusterrolebinding.rbac.authorization.k8s.io/kyverno-background-controller created|configured
deployment.apps/kyverno-background-controller created|configured
service/kyverno-background-controller created|configured
```

---

### Step 4: Verify Pods are Running
Ensure the pod reaches Ready (1/1) and logs show controllers started and caches synced.

```bash
# Pod should become 1/1 Running
kubectl get pods -n kyverno -l app=kyverno-background-controller

# Logs should show controllers starting and caches syncing
kubectl logs -n kyverno -l app=kyverno-background-controller --tail=100
```

Expected output (example):
```
NAME                                        READY   STATUS    RESTARTS   AGE
kyverno-background-controller-xxxxx         1/1     Running   0          30s
```

---

### Step 5: Verify Service and Access Metrics (HTTP on 8080)
Expose the metrics endpoint locally via port-forward and confirm it responds.

```bash
kubectl get svc -n kyverno kyverno-background-controller
kubectl get endpoints -n kyverno kyverno-background-controller
kubectl port-forward -n kyverno svc/kyverno-background-controller 8080:8080

# New terminal:
curl http://localhost:8080/metrics
```
Run this command in another terminal window to see metrics

---

## Troubleshooting

### Problem: CRDs missing or failed
Re-apply Kyverno v1.13 with server-side apply and re-verify the two required CRDs.

```bash
kubectl apply --server-side --force-conflicts -f https://github.com/kyverno/kyverno/releases/download/v1.13.0/install.yaml
kubectl get crd policies.kyverno.io
kubectl get crd clusterpolicies.kyverno.io
```

### Problem: Deployment apply error ("spec.selector is immutable")
Delete the existing Kyverno deployment first, then apply this repo’s deployment.

```bash
kubectl -n kyverno delete deployment kyverno-background-controller
kubectl apply -f deployment.yaml
```

### Problem: Pod 0/1 due to probe failures

Ensure probes use port 8080. This repo's `deployment.yaml` already sets readiness/liveness tcpSocket on port 8080.

### Problem: "Metrics API not available" for kubectl top

Install metrics-server and apply the TLS flags above (kind/minikube).

---

## Cleanup
Remove all resources created by this repo; optionally remove the Kyverno namespace.

```bash
kubectl delete -f deployment.yaml

# Delete this deployment
```

---