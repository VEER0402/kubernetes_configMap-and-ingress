
---

## 🎯 Why ConfigMaps & Secrets?

In real DevOps, **code and configuration must be separated**. This allows:

* Same Docker image across environments (dev / stage / prod)
* Secure handling of credentials
* Configuration changes without rebuilding images

This follows the **Twelve‑Factor App – Factor #3: Config**.

> *Store config in the environment, not in code.*

---

## 🧠 What is a ConfigMap?

A **ConfigMap** stores **non‑sensitive configuration** such as:

* Environment name (dev / prod)
* API URLs
* Feature flags
* Log levels

### Example

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: prod
  LOG_LEVEL: debug
```

---

## 🔐 What is a Secret?

A **Secret** stores **sensitive data** like:

* Database passwords
* API keys
* Tokens

> ⚠️ Kubernetes Secrets are **Base64 encoded**, not encrypted by default.

### Example

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: bXlwYXNzd29yZA==
```

---

## 🧪 How ConfigMaps & Secrets are Used in Pods

There are **two ways** to consume them:

---

### 1️⃣ Environment Variables (ENV)

#### Pod Example

```yaml
env:
- name: APP_ENV
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_ENV

- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

#### Behavior

* Values are read **only at pod startup**
* ConfigMap/Secret update ❌ does **NOT** reflect automatically
* Requires **rollout restart**

---

### 2️⃣ Volume Mount (File‑based)

#### Pod Example

```yaml
volumes:
- name: config-vol
  configMap:
    name: app-config

containers:
- name: app
  image: nginx
  volumeMounts:
  - name: config-vol
    mountPath: /etc/config
```

#### Behavior

* ConfigMap updates → files auto update
* App must support **reload / file watch**
* Kubernetes does **NOT** restart pods automatically

---

## 🔄 ENV vs Volume – Comparison

| Aspect                         | ENV         | Volume              |
| ------------------------------ | ----------- | ------------------- |
| Data type                      | Variable    | File                |
| Update reflected automatically | ❌ No        | ⚠️ App dependent    |
| Pod restart needed             | ✅ Yes       | ❌ Not always        |
| Best for                       | Flags, URLs | Config files, certs |

---

## 🔁 `kubectl rollout restart` – Why & When

### Command

```bash
kubectl rollout restart deployment <deployment-name>
```

### When to Use

* ConfigMap / Secret updated
* App consumes config via **ENV variables**
* Need safe restart **without downtime**

### What It Does

* Creates a new ReplicaSet
* Restarts pods **one by one**
* Ensures availability during restart

---

## 💥 Real Production Failure Story

### Situation

* App reads DB password from Secret via ENV
* Secret updated after password rotation

### Problem

* Pods continued using old password
* Application failures in production

### Root Cause

* ENV variables are read only at pod startup

### Fix

```bash
kubectl rollout restart deployment backend
```

---

## ☁️ How Secrets Are Used in EKS (Production)

In real EKS setups:

* Secrets are **not stored directly in Git**
* AWS Secrets Manager is used
* Synced into Kubernetes via External Secrets

Flow:

```
AWS Secrets Manager → Kubernetes Secret → Pod
```

---

## 🎤 Interview‑Ready Answers

### Q: When do you use rollout restart?

> *Whenever ConfigMaps or Secrets change and the application uses them via environment variables, we trigger a rollout restart so new pods pick up the updated values.*

---

### Q: Does ConfigMap update restart pods automatically?

> *No. Kubernetes never restarts pods automatically. Env‑based configs require restart; volume‑based configs depend on app reload capability.*

---

### Q: ENV vs Volume – when to use which?

> *ENV is used for simple configuration flags and URLs, while volume mounts are preferred for file‑based configs like nginx.conf or certificates.*

---

## 🧠 Key Takeaways

* Kubernetes **never auto‑restarts pods** for config changes
* ENV = static at startup
* Volume = dynamic file
* Rollout restart is **safe and production‑grade**
* ConfigMaps & Secrets implement **Twelve‑Factor App principles**

---

## 📌 Final Interview One‑Liner

> *ConfigMaps and Secrets help us follow the twelve‑factor app principle by separating configuration from code, and in EKS we integrate them with AWS Secrets Manager for secure production usage.*

---

✅ This README can be used for:

* Interview revision
* GitHub portfolio
* Kubernetes production reference
