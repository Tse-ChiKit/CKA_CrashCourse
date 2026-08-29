# Day 3 Troubleshooting — ConfigMap and Secret References

## Standard troubleshooting path

When a Pod cannot start because of configuration references:

```text
kubectl get pod
  -> inspect STATUS

kubectl describe pod <pod>
  -> inspect Events

Check in this order:
  1. referenced ConfigMap/Secret object name
  2. referenced key
  3. namespace
```

## Case 1 — ConfigMap object does not exist

Intentional bad reference:

```yaml
configMapKeyRef:
  name: web-config-wrong
  key: APP_ENV
```

Observed Event:

```text
spec.containers{nginx}: Error: configmap "web-config-wrong" not found
```

Meaning:

- Kubernetes cannot find the referenced ConfigMap object.
- First check the object name and namespace.

Useful commands:

```bash
kubectl get cm
kubectl get cm web-config-wrong
kubectl get cm web-config-wrong -n <namespace>
```

## Case 2 — ConfigMap exists but key does not

Intentional bad key:

```yaml
configMapKeyRef:
  name: web-config
  key: APP_ENV_WRONG
```

Observed Event:

```text
spec.containers{nginx}: Error: couldn't find key APP_ENV_WRONG in ConfigMap default/web-config
```

Meaning:

- `default/web-config` exists.
- The requested `APP_ENV_WRONG` key is missing.

Useful command:

```bash
kubectl get cm web-config -o yaml
```

Then inspect `data:` for the exact key name.

## Case 3 — Understand "not found" vs "couldn't find key"

These error messages identify different failure levels:

```text
configmap "NAME" not found
secret "NAME" not found
```

means:

```text
object lookup failed
```

while:

```text
couldn't find key KEY in ConfigMap ...
couldn't find key KEY in Secret ...
```

means:

```text
object exists, but key lookup failed
```

Do not jump directly to checking keys when the Event says the Secret/ConfigMap object itself is not found.

## Case 4 — Namespace mismatch

ConfigMap and Secret are namespaced resources.

A Pod in `test` cannot directly reference `default/web-config` merely by writing:

```yaml
name: web-config
```

The Pod searches its own namespace.

Check:

```bash
kubectl get pod <pod> -o wide
kubectl get pod <pod> -o jsonpath='{.metadata.namespace}'
kubectl get cm <configmap> -n <namespace>
kubectl get secret <secret> -n <namespace>
```

## Exam habit

For a configuration-related Pod startup failure, avoid guessing. `kubectl describe pod` Events usually state whether the problem is:

- missing ConfigMap/Secret
- missing key
- wrong namespace/object lookup

Read the exact Event message before editing YAML.
