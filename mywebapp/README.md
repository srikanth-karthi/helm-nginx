# MyWebApp Helm Chart

A Helm chart for deploying a custom Nginx web application to Kubernetes.

## Features

* Custom HTML homepage via ConfigMap
* Configurable replicas
* Health checks (liveness and readiness probes)
* Easy parameterization for multiple environments

## Installation

```bash
helm install mywebapp ./mywebapp
```

## Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `2` |
| `image.repository` | Docker image repository | `nginx` |
| `image.tag` | Docker image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `config.homepage` | Custom HTML homepage content | `<h1>Welcome to My WebApp!</h1>...` |

## Examples

### Install with custom homepage

```bash
helm install mywebapp ./mywebapp \
  --set config.homepage="<h1>Production Site</h1>"
```

### Install with 3 replicas

```bash
helm install mywebapp ./mywebapp \
  --set replicaCount=3
```

### Upgrade deployment

```bash
helm upgrade mywebapp ./mywebapp \
  --set config.homepage="<h1>Updated!</h1>"
```

### Rollback to previous version

```bash
helm rollback mywebapp 1
```

## Uninstallation

```bash
helm uninstall mywebapp
```

## Testing

Verify the chart:

```bash
helm lint ./mywebapp
```

Dry run to see generated manifests:

```bash
helm install mywebapp ./mywebapp --dry-run --debug
```
