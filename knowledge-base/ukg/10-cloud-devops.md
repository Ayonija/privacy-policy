# 10 — Cloud & DevOps

> Includes the asked **Docker**, **Kubernetes (pods/ingress/services)**, **CI/CD & deployments**, **Git**. Breadth topic — be confident and concrete, not exhaustive.

---

## 10.1 Docker (asked)
**Plain English:** Docker **packages an app + all its dependencies into a portable image** that runs the same anywhere. A **container** is a running instance of that image.

**How (define the terms):**
- **Image** — an immutable, layered snapshot (built from a `Dockerfile`). Layers are cached → fast rebuilds.
- **Container** — a running, isolated process using the host kernel (namespaces for isolation, cgroups for resource limits). **Lighter than a VM** — no guest OS, shares the host kernel → starts in ms, not minutes.
- **VM vs container:** VM virtualizes hardware (full guest OS, heavy); container virtualizes the OS (process isolation, light). Containers = density + speed; VMs = stronger isolation.
- **Registry** — stores images (Docker Hub, ECR, Artifactory).

**Best practices to name:** small base images (alpine/distroless), **multi-stage builds** (build in one stage, copy only the artifact to a slim runtime stage), `.dockerignore`, non-root user, pin versions, one process per container.

🎤 *"Docker packages an application with all its dependencies into a layered, immutable image that runs identically on my laptop, CI, and prod — it kills 'works on my machine.' A container is just that image running as an isolated process sharing the host kernel, which is why it's far lighter than a VM: no guest OS, so it starts in milliseconds. In practice I use multi-stage builds — compile in a fat stage, copy only the jar into a slim runtime image — and run as non-root, to keep images small and secure."*

---

## 10.2 Kubernetes (asked — pods, services, ingress)
**Plain English:** Kubernetes (K8s) is a **container orchestrator** — it runs, scales, heals, and networks containers across a cluster of machines, declaratively.

**Core objects (define each — these are the asked ones):**
- **Pod** — smallest deployable unit; one (usually) container + its shared network/storage. Ephemeral — pods die and get replaced (so never rely on a pod's IP).
- **Deployment** — declares the desired state ("run 3 replicas of this image"); the controller maintains it, does **rolling updates** and rollbacks. **ReplicaSet** underneath keeps N pods alive.
- **Service** — a **stable network endpoint + load balancer** for a set of pods (selected by labels), since pod IPs change. Types: **ClusterIP** (internal only), **NodePort**, **LoadBalancer** (cloud LB).
- **Ingress** — **L7 HTTP router** at the cluster edge: maps hostnames/paths to services, handles TLS. One entry point for many services (vs a LoadBalancer per service).
- **ConfigMap / Secret** — externalized config / sensitive values injected into pods.
- **HPA (Horizontal Pod Autoscaler)** — auto-scales replica count based on CPU/memory/custom metrics.
- **Namespace** — virtual cluster for isolation/multi-tenancy.

**Self-healing:** K8s restarts crashed containers, reschedules pods off dead nodes, and uses **liveness** (restart if unhealthy) + **readiness** (don't send traffic until ready) probes.

🎤 *"Kubernetes is declarative container orchestration — I describe the desired state and it converges to it, scaling and self-healing. The objects that matter: a pod is the smallest unit, one or more containers sharing a network, and it's ephemeral, so I never depend on its IP. A Deployment keeps N replicas alive and does rolling updates. Because pod IPs churn, a Service gives a stable virtual IP and load-balances across the matching pods, and an Ingress sits at the edge as an L7 router mapping hostnames and paths to services with TLS. For scale, the Horizontal Pod Autoscaler adds replicas on CPU or custom metrics, and readiness and liveness probes make sure traffic only goes to healthy pods."*

**Follow-up:** *Q: Rolling update vs blue-green vs canary?* (see 10.3) *Q: Service vs Ingress?* Service = L4 stable endpoint inside the cluster; Ingress = L7 HTTP routing at the edge across many services.

---

## 10.3 CI/CD & deployment strategies (asked)
- **CI (Continuous Integration)** — every commit is automatically built + tested (+ scanned: SonarQube/SAST) → catch breakage early, keep main releasable.
- **CD (Continuous Delivery/Deployment)** — automatically ship the validated artifact to environments; *delivery* = ready-to-deploy with a manual gate, *deployment* = fully automated to prod.
- **Pipeline stages:** checkout → build → unit test → static analysis/security scan → package (Docker image) → push to registry → deploy to staging → integration/E2E → promote to prod.
- **Deployment strategies (define each):**
  - **Rolling** — replace instances gradually (K8s default); no downtime, brief mixed-version window.
  - **Blue-green** — two full environments; switch traffic from blue→green instantly; easy rollback (switch back). Costs double infra briefly.
  - **Canary** — release to a small % of users first, watch metrics, then ramp. Safest for risky changes.
  - **Feature flags** — decouple deploy from release; turn features on/off at runtime without redeploy.

🎤 *"CI means every commit is automatically built, tested, and scanned so main stays releasable; CD means that validated artifact flows to environments automatically, with a gate before prod if needed. My pipeline builds a Docker image, runs the test pyramid plus SonarQube and dependency scanning, pushes to a registry, and deploys. For risky changes I prefer canary — release to a small slice, watch error rates and latency, then ramp — or blue-green when I want instant rollback by flipping traffic. And I separate deploy from release with feature flags, so shipping code and exposing a feature are independent decisions."*

---

## 10.4 Cloud essentials (Azure/AWS/GCP) — breadth
- **Compute:** VMs (EC2 / Azure VM / GCE), containers (EKS/AKS/GKE), serverless (Lambda / Functions / Cloud Functions).
- **Storage:** object (S3 / Blob / GCS), block, managed DB (RDS / Azure SQL / Cloud SQL).
- **Networking:** VPC, load balancers, CDN (CloudFront), DNS (Route 53).
- **Core principles:** **IAM** (least-privilege identity/access), **managed services over self-hosted** (offload ops), **multi-AZ** for availability, **auto-scaling**, **infrastructure-as-code** (Terraform/CloudFormation — reproducible, version-controlled infra). *You don't need provider trivia — speak to the patterns.*

---

## 10.5 Git & version control (asked)
- **Distributed VCS** — every clone is a full repo with history.
- **Branching:** **trunk-based** (short-lived branches, merge to main often, behind flags — pairs with CI/CD) vs **Git Flow** (long-lived develop/release branches — heavier).
- **merge vs rebase:** *merge* preserves history with a merge commit (truthful, can be noisy); *rebase* replays your commits onto the target for a linear history (clean, but rewrites commits — never rebase shared/pushed branches).
- **Other:** `cherry-pick` (grab one commit), `revert` (undo via a new commit — safe on shared branches) vs `reset` (move the branch pointer — rewrites, local only), pull requests + review as the quality gate, conflict resolution.

🎤 *"I favor trunk-based development with short-lived feature branches and frequent merges to main behind feature flags, because it keeps integration continuous and pairs naturally with CI/CD — long-lived branches just defer painful merges. On merge versus rebase: I rebase locally to keep a clean linear history before opening a PR, but I never rebase a branch others have pulled, because it rewrites commits. And to undo something already on a shared branch I use revert, which adds an inverse commit, rather than reset, which rewrites history."*

---

## ⚠️ Pitfalls & seniority signals
- Confusing container vs VM (kernel sharing), or Service vs Ingress (L4 vs L7).
- "We just deploy to prod" — no canary/rollback story.
- Rebasing shared branches; force-pushing main.
- **Seniority signal:** multi-stage Docker + non-root, readiness/liveness probes, canary + feature-flags (deploy ≠ release), IaC + least-privilege IAM, trunk-based + CI/CD coherence.

---

*Next topic, or drill deeper on this one?*
