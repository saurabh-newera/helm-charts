# Helm Charts Repository

This repository contains Helm charts for deploying microservices in Kubernetes clusters. Charts are automatically published to GitHub Pages and used by Fleet for GitOps deployments.

---

##  Available Charts

| Chart | Description | Current Version | App Version |
|-------|-------------|-----------------|-------------|
| **nginx-service** | Nginx web server | 0.3.0 | 1.27.0 |
| **httpbin-service** | HTTP testing service | 0.3.0 | latest |
| **redis-service** | Redis cache | 0.3.0 | 7.2.0 |
| **postgres** | PostgreSQL database | 1.2.0 | 16 |
| **store-stack** | Umbrella chart (all services) | 0.1.0 | - |

---

##  Repository Structure

```
helm-charts/
│
├── .github/
│   └── workflows/
│       └── release-charts.yaml     # Automated chart publishing
│
├── k8s/
│   ├── Chart.yaml                  # store-stack umbrella chart
│   │
│   ├── nginx-service/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       └── _helpers.tpl
│   │
│   ├── httpbin-service/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │
│   ├── redis-service/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │
│   └── postgres/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│
└── README.md
```

---

##  Automated Chart Publishing

This repository uses **GitHub Actions** to automatically package and publish Helm charts to **GitHub Pages** whenever changes are pushed to the `main` branch.

### How It Works

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     AUTOMATED CHART PUBLISHING FLOW                     │
│                                                                         │
│   Developer                                                             │
│       │                                                                 │
│       │  1. Update Chart.yaml version                                  │
│       │  2. Modify templates or values                                 │
│       │  3. Commit & push to main                                      │
│       │                                                                 │
│       ▼                                                                 │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  GitHub Actions (.github/workflows/release-charts.yaml)         │  │
│   │                                                                 │  │
│   │  Step 1: Checkout code                                          │  │
│   │  Step 2: Install Helm                                           │  │
│   │  Step 3: Run chart-releaser                                     │  │
│   │          • Package charts (helm package)                        │  │
│   │          • Generate index.yaml                                  │  │
│   │          • Create GitHub Release                                │  │
│   │          • Commit to gh-pages branch                            │  │
│   └───────────────────────────────┬─────────────────────────────────┘  │
│                                   │                                     │
│                                   ▼                                     │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  GitHub Pages (gh-pages branch)                                 │  │
│   │                                                                 │  │
│   │  • index.yaml (Helm repository index)                           │  │
│   │  • nginx-service-0.3.0.tgz                                      │  │
│   │  • httpbin-service-0.3.0.tgz                                    │  │
│   │  • redis-service-0.3.0.tgz                                      │  │
│   │  • postgres-1.2.0.tgz                                           │  │
│   └───────────────────────────────┬─────────────────────────────────┘  │
│                                   │                                     │
│                                   │ Publicly accessible at:             │
│                                   │ https://saurabh-newera.github.io/   │
│                                   │        helm-charts/                 │
│                                   ▼                                     │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  Users / Fleet / Helm CLI                                       │  │
│   │                                                                 │  │
│   │  helm repo add myrepo https://saurabh-newera.github.io/...     │  │
│   │  helm install my-app myrepo/nginx-service                       │  │
│   └─────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

### GitHub Actions Workflow

The workflow (`.github/workflows/release-charts.yaml`) triggers on:
- **Push to main** branch with changes in `k8s/**`
- **Manual trigger** via GitHub UI (workflow_dispatch)

**What it does:**
1.  Checks out the repository
2.  Installs Helm
3.  Creates `gh-pages` branch if it doesn't exist
4.  Runs [chart-releaser](https://github.com/helm/chart-releaser) to:
   - Package each chart into `.tgz` files
   - Generate `index.yaml` (Helm repository index)
   - Create GitHub Releases with chart artifacts
   - Push to `gh-pages` branch

### Versioning Strategy

Charts follow **Semantic Versioning** (semver):
- **MAJOR**: Breaking changes (e.g., `1.0.0` → `2.0.0`)
- **MINOR**: New features, backwards-compatible (e.g., `0.2.0` → `0.3.0`)
- **PATCH**: Bug fixes (e.g., `0.3.0` → `0.3.1`)

**Important:** To publish a new chart version:
1. Bump the `version` field in `Chart.yaml`
2. Commit and push to `main`
3. GitHub Actions automatically publishes it

---

##  Using Published Charts

### Add Helm Repository

```bash
helm repo add saurabh-helm https://saurabh-newera.github.io/helm-charts
helm repo update
```

### Search Available Charts

```bash
helm search repo saurabh-helm
```

Expected output:
```
NAME                          CHART VERSION   APP VERSION   DESCRIPTION
saurabh-helm/nginx-service    0.3.0          1.27.0        A simple Nginx web server
saurabh-helm/httpbin-service  0.3.0          latest        HTTP testing service
saurabh-helm/redis-service    0.3.0          7.2.0         Redis cache
saurabh-helm/postgres         1.2.0          16            PostgreSQL database
```

### Install a Chart

```bash
# Install with default values
helm install my-nginx saurabh-helm/nginx-service

# Install with custom values
helm install my-nginx saurabh-helm/nginx-service \
  --set replicaCount=3 \
  --set image.tag="1.25.0"

# Install with values file
helm install my-nginx saurabh-helm/nginx-service \
  -f my-values.yaml
```

### Upgrade a Release

```bash
helm upgrade my-nginx saurabh-helm/nginx-service --version 0.3.0
```

### Uninstall a Release

```bash
helm uninstall my-nginx
```

---

##  Local Development

### Prerequisites

- [Helm](https://helm.sh/docs/intro/install/) v3.x installed
- kubectl configured (for testing installations)

### Linting Charts

```bash
cd k8s/nginx-service
helm lint .
```

### Testing Chart Installation Locally

```bash
# Dry run (renders templates without installing)
helm install test-nginx ./k8s/nginx-service --dry-run --debug

# Install from local directory
helm install test-nginx ./k8s/nginx-service

# Uninstall
helm uninstall test-nginx
```

### Package Chart Manually

```bash
cd k8s
helm package nginx-service/
# Creates: nginx-service-0.3.0.tgz
```

### Update Dependencies (for umbrella charts)

```bash
cd k8s
helm dependency update
```

---

##  Using with Fleet (GitOps)

These charts are designed to work with **Fleet by Rancher** for multi-cluster GitOps deployments.

### In GitOps Repository

Reference these published charts in your `Chart.yaml`:

```yaml
# gitops-fleet/apps/store-stack/Chart.yaml
apiVersion: v2
name: store-stack
version: 1.0.0
dependencies:
  - name: nginx-service
    version: 0.3.0
    repository: https://saurabh-newera.github.io/helm-charts
  
  - name: redis-service
    version: 0.3.0
    repository: https://saurabh-newera.github.io/helm-charts
  
  - name: httpbin-service
    version: 0.3.0
    repository: https://saurabh-newera.github.io/helm-charts
  
  - name: postgres
    version: 1.2.0
    repository: https://saurabh-newera.github.io/helm-charts
```

Then Fleet will:
1. Pull charts from the published Helm repository
2. Merge with environment-specific values
3. Deploy to target clusters

**See [gitops-fleet-2](https://github.com/saurabh-newera/gitops-fleet-2) for complete GitOps configuration examples.**

---

##  Chart Development Guidelines

### Creating a New Chart

```bash
cd k8s
helm create my-new-service
```

### Chart.yaml Requirements

```yaml
apiVersion: v2
name: my-service
description: Brief description of the service
type: application
version: 0.1.0           # Chart version (bump on every change)
appVersion: "1.0.0"      # Application version
maintainers:
  - name: Your Name
    email: your-email@example.com
```

### values.yaml Best Practices

```yaml
# Use sensible defaults that work for local dev
replicaCount: 1

image:
  repository: myapp
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

**Note:** Production values (replicas, resources, IPs) should be overridden in the GitOps repository, not here.

---

##  Permissions & Secrets

### GitHub Actions Permissions

The workflow requires:
- `contents: write` - To create releases and push to gh-pages
- `packages: write` - For potential OCI registry support

These are configured in `.github/workflows/release-charts.yaml`.

### GitHub Pages Setup

1. Go to **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: **gh-pages** / **(root)**
4. Save

GitHub Pages will be available at:
`https://<username>.github.io/<repository-name>/`

---

##  Troubleshooting

### Chart Not Publishing

**Check workflow run:**
1. Go to **Actions** tab in GitHub
2. Click on latest "Release Charts" workflow
3. Review logs for errors

**Common issues:**
-  Version already exists: Bump version in `Chart.yaml`
-  gh-pages branch missing: Workflow auto-creates it
-  Syntax errors: Run `helm lint` locally first

### Chart Not Found After Publishing

```bash
# Wait 2-3 minutes for GitHub Pages to update
# Then update your local repo index
helm repo update

# Verify it's in the index
curl https://saurabh-newera.github.io/helm-charts/index.yaml
```

### Testing Before Publishing

```bash
# Lint all charts
cd k8s
for dir in */; do
  if [ -f "$dir/Chart.yaml" ]; then
    echo "Linting $dir"
    helm lint "$dir"
  fi
done

# Test packaging
helm package nginx-service/
helm package redis-service/
helm package httpbin-service/
helm package postgres/
```

---

##  Chart Release History

| Version | Date | Changes |
|---------|------|---------|
| 0.3.0 | Jan 2026 | Bumped nginx, httpbin, redis versions |
| 1.2.0 | Jan 2026 | Updated postgres chart |
| 0.2.0 | Dec 2025 | Initial automated publishing setup |
| 0.1.0 | Dec 2025 | Initial chart creation |

---

##  Contributing

1. Create a feature branch
2. Make changes to chart(s)
3. Update `version` in `Chart.yaml`
4. Test locally with `helm lint` and `helm install --dry-run`
5. Create PR to `main`
6. After merge, GitHub Actions automatically publishes

---

##  Additional Resources

- [Helm Documentation](https://helm.sh/docs/)
- [Chart Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Chart Releaser Action](https://github.com/helm/chart-releaser-action)
- [Fleet by Rancher](https://fleet.rancher.io/)
- [GitOps Fleet Configuration](https://github.com/saurabh-newera/gitops-fleet-2)

---

##  License

MIT License - see individual chart directories for details.

---

**Helm Repository**: https://saurabh-newera.github.io/helm-charts  
**GitOps Configuration**: https://github.com/saurabh-newera/gitops-fleet-2
