# CKA Crash Course

A 3-week hands-on study repository for the Certified Kubernetes Administrator (CKA) exam.

## Goals

- Understand Kubernetes architecture and core component responsibilities.
- Build and operate a real 3-node kubeadm cluster.
- Learn through hands-on labs, failure injection, and troubleshooting.
- Prepare for the CKA performance-based exam with timed mock exercises.
- Keep daily study history separate from reusable long-term notes.

## Repository structure

```text
CKA_CrashCourse/
├── daily/          # Daily plans, execution notes, questions and troubleshooting
├── concepts/       # Reusable Kubernetes concept notes
├── labs/           # Hands-on lab procedures and manifests
├── project/        # FastAPI/Agent + Redis/Postgres + Helm project
├── mock-exams/     # Timed CKA-style mock exams
└── cheatsheets/    # Commands and troubleshooting quick references
```

## Study workflow

Each day has two dimensions:

1. **Planned learning** — goals, concepts, commands and lab tasks.
2. **Actual learning record** — commands executed, questions, observations, errors, fixes and exam takeaways.

Important knowledge is later promoted from the daily log into `concepts/` or `cheatsheets/` so the repository does not become a collection of daily transcripts.

## Progress

- [x] Day 0 — Prepare Linux nodes and Kubernetes prerequisites
- [x] Day 1 — Kubernetes architecture, kubeadm bootstrap, Calico CNI, worker join and cluster troubleshooting
- [x] Day 2 — Pods, Deployments, ReplicaSets, rollout/rollback, Services, labels/selectors and declarative YAML
- [x] Day 3 — ConfigMap, Secret, environment variables, configuration Volumes and reference troubleshooting
- [ ] Day 4 — Workload reliability and scheduling topics

## Current lab checkpoint

The lab has a working 3-node kubeadm cluster:

```text
k8s-control
k8s-worker-01
k8s-worker-02
```

The control-plane was initialized with kubeadm, Calico is installed as the CNI, and both worker nodes have joined the cluster.

### Day 2 workload administration

- bare Pods and Pod lifecycle inspection
- Deployments and ReplicaSets
- scaling and self-healing
- RollingUpdate and rollback
- ClusterIP and NodePort Services
- label/selector based backend discovery
- Endpoint/EndpointSlice inspection
- declarative Deployment YAML and `kubectl apply`

### Day 3 configuration management

- ConfigMap creation and inspection
- `configMapKeyRef` and `envFrom/configMapRef`
- ConfigMap Volume mounts and key-to-file mapping
- environment-variable vs Volume update behavior
- ConfigMap/Secret namespace scope
- generic Opaque Secrets
- Base64 encoding vs encryption
- `secretKeyRef`, `secretRef` and Secret Volume patterns
- troubleshooting missing objects vs missing keys with Pod Events
- YAML `|` multiline literal syntax

Day 4 can continue with workload reliability and scheduling topics such as probes, resource requests/limits and related troubleshooting.
