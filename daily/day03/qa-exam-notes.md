# Day 3 Q&A and Exam Notes

## Q1 — Why use ConfigMap instead of hardcoding values in Pod YAML?

User's initial reasoning:

> Pod may be destroyed and recreated, so if config is hardcoded in the Pod it may be lost. The biggest value is decoupling Pod and config.

Correction:

A Deployment-managed Pod recreated from the same Pod template normally keeps hardcoded values. The stronger reason is **decoupling configuration from the workload definition**, making configuration independently manageable and reusable.

## Q2 — Is ConfigMap lookup based only on name?

No. ConfigMap is namespaced.

A Pod can directly reference a ConfigMap only in the same namespace. A reference such as `name: web-config` is resolved inside the Pod's namespace.

## Q3 — If YAML omits namespace, is it `default`?

Usually yes in the standard default kubectl context, but more precisely: kubectl uses the namespace from the active context or the `-n` flag. If no namespace is configured, that resolves to `default`.

## Q4 — What is the difference between `configMapKeyRef` and `configMapRef`?

```text
configMapKeyRef -> choose one specific key
configMapRef    -> import the whole ConfigMap with envFrom
```

If the Pod should receive `APP_ENV` but not `LOG_LEVEL`, use `configMapKeyRef`.

## Q5 — What happens when ConfigMap is mounted as a Volume?

For:

```yaml
data:
  config.txt: hello from configmap
```

mounted at:

```text
/etc/config
```

Kubernetes presents:

```text
/etc/config/config.txt
```

with contents:

```text
hello from configmap
```

Rule:

```text
key -> filename
value -> file contents
```

## Q6 — Environment variable vs Volume ConfigMap update

Verified experimentally:

- environment variable value remained `production` after ConfigMap changed to `staging`
- mounted ConfigMap file later changed to `hello from updated configmap`

Exam takeaway:

```text
env/envFrom -> running container does not receive changed environment variable automatically
ConfigMap Volume -> projected file can refresh after a delay
```

Application reload behavior is separate from Kubernetes updating the file.

## Q7 — Is Kubernetes Secret data encrypted because it looks unreadable?

No. The observed values:

```text
YWRtaW4=
bXlzZWNyZXQxMjM=
```

are Base64 **encoded**, not encrypted.

This was an important correction during Day 3.

## Q8 — When a Secret is injected into a Pod, does the Pod receive the Base64 value?

No.

The initial prediction was that the Pod might receive:

```text
bXlzZWNyZXQxMjM=
```

but the lab proved Kubernetes provides the decoded value:

```text
mysecret123
```

## Q9 — Secret Volume path question

If:

```text
Secret key = DB_PASSWORD
mountPath  = /etc/secret
```

then the file path is:

```text
/etc/secret/DB_PASSWORD
```

## Q10 — Troubleshooting question

Event:

```text
secret "db-password" not found
```

The first thing to check is **whether the Secret object exists in the Pod's namespace**, not whether a particular key exists.

Compare:

```text
secret "db-password" not found
-> object missing / wrong namespace / wrong object name

couldn't find key PASSWORD in Secret ...
-> Secret exists, key missing
```

## CKA-style quiz results

### Question 1

A ConfigMap value used as an environment variable changes from `production` to `staging`. Does an already-running Pod immediately see `staging`?

**Answer:** No. ✅

### Question 2

ConfigMap key `nginx.conf` is mounted at `/etc/nginx/config`. What is the resulting path?

**Answer:** `/etc/nginx/config/nginx.conf` ✅

### Question 3

Which resource is more appropriate for a database password?

**Answer:** Secret ✅

### Question 4

Event says `secret "db-password" not found`. What should be checked first?

Initial answer: check whether the Secret key exists.

Correction: first check whether the **Secret object itself exists in the correct namespace**. ⚠️

## Fast exam memory map

```text
ConfigMap -> non-sensitive configuration
Secret    -> sensitive configuration

configMapKeyRef -> one ConfigMap key
configMapRef    -> whole ConfigMap
secretKeyRef    -> one Secret key
secretRef       -> whole Secret

env/envFrom -> environment variables
Volume      -> files

not found         -> check object name + namespace
couldn't find key -> check data key

Base64 -> encoding, not encryption
```

## YAML syntax note — `|`

```yaml
nginx.conf: |
  server {
    listen 80;
  }
```

`|` means the indented content is a literal multiline string and line breaks are preserved. This is common when embedding configuration files in YAML.
