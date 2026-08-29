# CKA Day 3 — ConfigMap, Secret and Pod Configuration

Date: 2026-08-29

## Day 3 goals

Learn how Kubernetes separates application configuration from workload definitions and how Pods consume non-sensitive and sensitive configuration.

## Completed

- [x] Understand why ConfigMap decouples configuration from Pod/Deployment definitions
- [x] Create ConfigMaps from literal values
- [x] Inspect ConfigMaps with `kubectl get`, `describe` and `-o yaml`
- [x] Inject one ConfigMap key with `configMapKeyRef`
- [x] Inject an entire ConfigMap with `envFrom` + `configMapRef`
- [x] Mount ConfigMap data as files through a Volume
- [x] Understand `volumes` vs `volumeMounts`
- [x] Observe ConfigMap updates through environment variables vs mounted files
- [x] Understand ConfigMap and Pod namespace requirements
- [x] Create an Opaque Secret
- [x] Understand Base64 encoding vs encryption
- [x] Inject Secret values into Pod environment variables with `secretKeyRef`
- [x] Understand Secret `envFrom` and Secret Volume patterns
- [x] Troubleshoot missing ConfigMap objects and missing keys
- [x] Review YAML literal block scalar `|`

## End-of-day mental model

```text
ConfigMap
  -> non-sensitive configuration
  -> can become environment variables or mounted files

Secret
  -> sensitive configuration
  -> data is commonly represented as Base64 in YAML
  -> Base64 is encoding, not encryption

Pod
  -> references ConfigMap/Secret in the same namespace
  -> env/configMapKeyRef or secretKeyRef = selected key
  -> envFrom/configMapRef or secretRef = whole object
  -> Volume = configuration appears as files
```

## Lab checkpoint

The cluster now has working examples of:

- `web-config`
- `config-demo`
- `config-demo-all`
- `app-file-config`
- `config-volume-demo`
- `db-secret`
- `secret-demo`
- intentional ConfigMap reference failures for troubleshooting

## Key Day 3 takeaway

Configuration should not be tightly coupled to workload YAML. ConfigMap and Secret let a workload reference separately managed configuration, while environment-variable and Volume consumption modes have different update behavior.
