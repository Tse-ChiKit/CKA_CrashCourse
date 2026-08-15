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
- [ ] Day 1 — Kubernetes architecture and bootstrap the real cluster
