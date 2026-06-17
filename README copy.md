# WordPress Local GitOps

A production-inspired local Kubernetes GitOps repository using FluxCD, NGINX Ingress, MetalLB, Prometheus, Grafana, Loki, Promtail, SOPS, and WordPress.

> **Looking for a production-ready deployment?**
>
> This repository is intended for local development, learning, testing, and homelab environments.
>
> For a production-focused implementation with additional hardening, operational best practices, and production-oriented GitOps configurations, please refer to:
>
> **Production Repository:** [wordpress-production](https://github.com/crchiran/wordpress-production-gitops.git?utm_source=chatgpt.com)

## Architecture

```text
Git Repository
      │
      ▼
   FluxCD
      │
      ├── Infrastructure
      │     ├── MetalLB
      │     ├── Ingress NGINX
      │     └── Observability
      │
      └── Applications
            └── WordPress
                  ├── MariaDB
                  └── WordPress
```

---

## Components

### Infrastructure

| Component     | Purpose                                               |
| ------------- | ----------------------------------------------------- |
| FluxCD        | GitOps continuous delivery                            |
| MetalLB       | LoadBalancer implementation for bare-metal Kubernetes |
| Ingress NGINX | Ingress controller                                    |
| Prometheus    | Metrics collection                                    |
| Grafana       | Monitoring dashboards                                 |
| Loki          | Log aggregation                                       |
| Promtail      | Log shipping                                          |

### Applications

| Component | Purpose            |
| --------- | ------------------ |
| WordPress | CMS application    |
| MariaDB   | WordPress database |

### Security

| Component         | Purpose                                        |
| ----------------- | ---------------------------------------------- |
| SOPS              | Secret encryption                              |
| Age               | Encryption key management                      |
| FluxCD Decryption | Automatic secret decryption inside the cluster |

---

## Repository Structure

```text
.
├── README.md
└── clusters
    └── local
        ├── apps
        │   ├── blog
        │   │   ├── wordpress
        │   │   ├── wp-database
        │   │   └── wp-namespace
        │   ├── blog-wp-mysql.yaml
        │   ├── blog-wp-ns.yaml
        │   ├── kustomization.yaml
        │   └── wp.yaml
        │
        ├── flux-system
        │
        ├── infrastructure
        │   ├── ingress-nginx
        │   ├── metallb
        │   └── observability
        │
        └── kustomization.yaml
```

---

## Application Layout

### Namespace

```text
apps/blog/wp-namespace
```

Creates:

```text
blog namespace
```

### Database

```text
apps/blog/wp-database
```

Deploys:

```text
MariaDB StatefulSet
MariaDB Service
Database ConfigMap
Encrypted Database Secret
```

Files:

```text
configmap.yaml
mysql-secret.enc.yaml
mysql-sts.yaml
mysql-svc.yaml
```

### WordPress

```text
apps/blog/wordpress
```

Deploys:

```text
WordPress Deployment
Persistent Volume Claim
Service
Ingress
Encrypted Application Secret
```

Files:

```text
wordpress-secret.enc.yaml
wp-deployment.yaml
wp-pvc.yaml
wp-svc.yaml
wp-ingress.yaml
```

---

## Infrastructure Layout

### MetalLB

```text
infrastructure/metallb
```

Provides:

```text
LoadBalancer services
L2 advertisements
IP address pools
```

### Ingress NGINX

```text
infrastructure/ingress-nginx
```

Provides:

```text
Ingress controller
LoadBalancer service
IngressClass
```

### Observability

```text
infrastructure/observability
```

Provides:

```text
Prometheus
Grafana
Loki
Promtail
Grafana Dashboards
```

---

## Secret Management

Secrets are encrypted using SOPS and Age.

### Encrypted Files

```text
mysql-secret.enc.yaml
wordpress-secret.enc.yaml
```

### SOPS Configuration

Repository root:

```text
.sops.yaml
```

Example:

```yaml
creation_rules:
  - path_regex: clusters/local/.*\.enc\.yaml$
    encrypted_regex: ^(data|stringData)$
    age: age1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Encrypt Secret

```bash
sops --encrypt --in-place \
clusters/local/apps/blog/wp-database/mysql-secret.enc.yaml
```

### Edit Secret

```bash
sops \
clusters/local/apps/blog/wp-database/mysql-secret.enc.yaml
```

### Decrypt Secret

```bash
sops --decrypt \
clusters/local/apps/blog/wp-database/mysql-secret.enc.yaml
```

---

## Bootstrap FluxCD

Install Flux:

```bash
flux bootstrap github \
  --owner=<github-user> \
  --repository=wordpress-local-gitops \
  --branch=main \
  --path=clusters/local \
  --personal
```

Verify:

```bash
flux get sources git
flux get kustomizations
```

---

## Deploy Infrastructure

Reconcile:

```bash
flux reconcile source git flux-system --with-source
```

Check:

```bash
flux get ks
```

Expected:

```text
infra-metallb
infra-ingress-nginx
infra-monitoring
```

---

## Deploy WordPress

Deploy order:

```text
1. Namespace
2. Database
3. WordPress
```

Verify:

```bash
kubectl get pods -n blog
```

Expected:

```text
mysql-0
wordpress-xxxxx
```

---

## Access WordPress

Check Ingress:

```bash
kubectl get ingress -n blog
```

Example:

```text
Host: app.local
```

Add host entry:

```bash
sudo nano /etc/hosts
```

```text
192.168.10.61 app.local
```

Access:

```text
http://app.local
```

---

## Monitoring

Access Grafana:

```text
http://grafana.local
```

Retrieve admin password:

```bash
kubectl get secret grafana-admin \
  -n monitoring \
  -o jsonpath='{.data.admin-password}' | base64 -d
```

---

## Useful Commands

### Flux

```bash
flux get ks
flux get hr
flux reconcile ks <name> --with-source
```

### Kubernetes

```bash
kubectl get pods -A
kubectl get pvc -A
kubectl get ingress -A
kubectl get svc -A
```

### Logs

```bash
kubectl logs -n blog deploy/wordpress
kubectl logs -n blog sts/mysql
```

### SOPS

```bash
sops file.enc.yaml
sops --decrypt file.enc.yaml
sops --encrypt --in-place file.enc.yaml
```

---

## Security Features

* GitOps deployment with FluxCD
* Encrypted secrets using SOPS and Age
* NGINX Ingress with forwarded headers
* Non-root containers where possible
* Persistent storage for WordPress and MariaDB
* Observability with Prometheus, Grafana, Loki, and Promtail
* Infrastructure and application separation

---

