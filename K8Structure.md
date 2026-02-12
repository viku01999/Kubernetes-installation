# 🏗 Big Picture Architecture

Here is the full structure:

```bash
Namespace: dev
│
├── Deployment
│     └── Pod
│           └── Container
│
├── ConfigMap  (Non-sensitive data)
│
└── Secret     (Sensitive data)
```

**When pod starts:**

```bash
Pod reads:
  → ConfigMap
  → Secret
```

Kubernetes injects them at runtime.

# 🟣 Volume Mount Method (Structure Level)

```bash
Pod
 └── Container
       └── /app/config/
             ENV
             LOG_LEVEL
       └── /app/secret/
             DB_PASSWORD
             JWT_SECRET
```

Deployment example:

```yaml
volumeMounts:
  - name: config-volume
    mountPath: /app/config
  - name: secret-volume
    mountPath: /app/secret

volumes:
  - name: config-volume
    configMap:
      name: node-app-config

  - name: secret-volume
    secret:
      secretName: node-app-secret
```

## 🏢 Real Production Folder Structure

```bash
k8s/
 ├── dev/
 │     ├── deployment.yaml
 │     ├── configmap.yaml
 │     ├── secret.yaml
 │
 ├── prod/
 │     ├── deployment.yaml
 │     ├── configmap.yaml
 │     ├── secret.yaml
```

## 🔐 Important Security Understanding

Secrets are:

- Base64 encoded
- Stored in etcd

For real production:
Use:

- AWS Secrets Manager
- External Secrets Operator
- Vault

## Final Structure Mental Model

```bash
Docker Image  = Application code
ConfigMap     = Application settings
Secret        = Sensitive credentials
Deployment    = Controls pods
HPA           = Scales pods
Service       = Exposes pods
```

## 🏗 Complete Project Structure (From Root)

```bash
project-root/
│
├── src/                        # Application source code
│   ├── index.ts
│   └── ...
│
├── Dockerfile
├── package.json
├── package-lock.json
│
├── .env                        # Local development only (NOT for k8s)
│
├── k8s/                        # Kubernetes main folder
│
│   ├── base/                   # Base reusable configs
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap.yaml
│   │   ├── secret.yaml
│   │   └── hpa.yaml
│   │
│   ├── dev/                    # Dev environment configs
│   │   ├── namespace.yaml
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap.yaml
│   │   ├── secret.yaml
│   │   └── hpa.yaml
│   │
│   ├── prod/                   # Production environment configs
│   │   ├── namespace.yaml
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap.yaml
│   │   ├── secret.yaml
│   │   ├── hpa.yaml
│   │   └── ingress.yaml
│   │
│   └── README.md               # Notes about k8s setup
│
└── README.md                   # Project documentation
```
