# store-stack

An umbrella Helm chart that bundles the complete store application stack.

## Components

This chart includes the following services:

- **nginx-service** (0.2.0) - Web server
- **redis-service** (0.2.0) - In-memory data store
- **httpbin-service** (0.2.0) - HTTP request/response testing service

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

## Installation

### Add the Helm repository

```bash
helm repo add store-charts https://saurabh-newera.github.io/helm-charts
helm repo update
```

### Install the chart

```bash
helm install my-store store-charts/store-stack --version 1.4.0
```

### Install with custom values

```bash
helm install my-store store-charts/store-stack \
  --version 1.4.0 \
  --set nginx-service.replicaCount=3 \
  --set redis-service.resources.limits.memory=512Mi
```

## Configuration

The chart supports all configuration options from its component charts. Values are namespaced by service:

### nginx-service configuration

```yaml
nginx-service:
  replicaCount: 2
  image:
    tag: "1.27.0"
  service:
    type: LoadBalancer
    loadBalancerIP: "10.100.1.10"
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
  env:
    NGINX_HOST: example.com
```

### redis-service configuration

```yaml
redis-service:
  replicaCount: 1
  image:
    tag: "7.2.0"
  resources:
    limits:
      memory: 256Mi
  env:
    REDIS_MAXMEMORY: "256mb"
```

### httpbin-service configuration

```yaml
httpbin-service:
  replicaCount: 1
  service:
    type: ClusterIP
  resources:
    limits:
      cpu: 100m
      memory: 128Mi
```

## Usage with Fleet

This chart is designed to work with Fleet GitOps for multi-cluster deployments:

```yaml
# fleet.yaml
helm:
  repo: https://saurabh-newera.github.io/helm-charts
  chart: store-stack
  version: 1.4.0

targetCustomizations:
  - name: store-cluster-001
    clusterSelector:
      matchLabels:
        cluster-name: store-001
    helm:
      valuesFiles:
        - versions/v1.4.0.yaml
        - clusters/store-001.yaml
```

## Upgrading

### From v1.4.0 to v1.5.0

```bash
helm upgrade my-store store-charts/store-stack --version 1.5.0
```

## Uninstalling

```bash
helm uninstall my-store
```

## Dependencies

This chart automatically pulls the following dependencies from the Helm repository:

| Dependency | Version | Repository |
|------------|---------|------------|
| nginx-service | 0.2.0 | https://saurabh-newera.github.io/helm-charts |
| redis-service | 0.2.0 | https://saurabh-newera.github.io/helm-charts |
| httpbin-service | 0.2.0 | https://saurabh-newera.github.io/helm-charts |

## Chart Versions

| Chart Version | nginx-service | redis-service | httpbin-service | Description |
|---------------|---------------|---------------|-----------------|-------------|
| 1.4.0 | 0.2.0 | 0.2.0 | 0.2.0 | Initial stable release |

## Support

For issues and questions, please open an issue in the GitHub repository.
