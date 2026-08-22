# SRE Production Lab

A self-directed, hands-on project simulating real production SRE/DevOps work — provisioning infrastructure, deploying a real multi-service application, and diagnosing genuine incidents along the way.

## What This Is

This repo documents my hands-on journey building and operating a production-style environment, using [Google's Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) (a real 10-service e-commerce application) as the system under management. The goal: build genuine, defensible production experience — not just complete tutorials.

## Stack

- **Local Kubernetes:** kind
- **Application:** Online Boutique (10-service microservices demo, gRPC-based)
- **Networking:** nginx Ingress Controller
- *(Coming: Terraform, ArgoCD, Istio, Prometheus/Grafana, Chaos Engineering)*

## What's Been Built So Far

- Local Kubernetes cluster (`kind`) running a real 12-Pod microservices application
- Hands-on Deployment/Service/self-healing verification (manual Pod kills, scaling tests)
- nginx Ingress Controller installed and configured, routing real external traffic through an Ingress rule to the frontend Service
- AWS account configured with least-privilege IAM access (no root usage) and billing safety alarms

## Incidents Diagnosed & Resolved

### 1. `cartservice` — OOMKilled / CrashLoopBackOff
**Symptom:** Storefront homepage returned HTTP 500; frontend logs showed `connection refused` to cartservice.
**Diagnosis:** `kubectl describe pod` revealed `Last State: Terminated, Reason: OOMKilled, Exit Code: 137` — the container's memory usage exceeded its configured limit (128Mi), triggering a Linux kernel OOM kill, followed by a Kubernetes crash-loop as it repeatedly retried.
**Fix:** Raised the Deployment's memory limit/request (`kubectl set resources`), triggering a rolling update to a new, stable Pod.
**Root cause:** Default resource limits were tuned for larger cloud infrastructure, not a local laptop-based cluster.

### 2. `currencyservice` — Flaky liveness/readiness probe timeouts
**Symptom:** Repeated restarts (23x) despite the service being fundamentally healthy.
**Diagnosis:** Liveness/readiness probes were configured with an aggressive 1-second timeout; under local resource contention (shared laptop CPU vs. dedicated cloud infra), the container occasionally failed to respond within that window, triggering false-positive restarts.
**Learning:** A working service can still generate a stream of restarts if health checks are miscalibrated for the environment — a common real-world false-positive pattern, not unique to local clusters.

### 3. `productcatalogservice` — Deliberate failure injection (control case)
**Symptom:** N/A — manually deleted the Pod to observe self-healing behavior.
**Result:** Recreated and stable within ~5 seconds, no further issues.
**Purpose:** Used as a contrast case against incident #1 — demonstrating the difference between a *transient* failure (self-healing resolves it completely) and a *systemic* failure (self-healing alone can't fix a misconfiguration).

## Next Steps

- Provision infrastructure via Terraform (local → AWS EKS)
- Deploy via ArgoCD (GitOps)
- Add Istio service mesh (traffic management, mTLS)
- Add Prometheus/Grafana observability
- Chaos engineering experiments + written postmortems
