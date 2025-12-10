# PainChain Helm Chart

Officially supported Helm chart for PainChain - a change tracking and visualization platform.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- PersistentVolume provisioner support in the underlying infrastructure (for PostgreSQL data persistence)

## Installation

### Quick Start

Install PainChain with default configuration:

```bash
helm install painchain .
```

### Custom Installation

Install with custom values:

```bash
helm install painchain . -f custom-values.yaml
```

### Install with custom database password:

```bash
helm install painchain . --set postgresql.auth.password=your-secure-password
```

## Configuration

The following table lists the configurable parameters of the PainChain chart and their default values.

### Global Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.imagePullPolicy` | Image pull policy | `IfNotPresent` |

### PostgreSQL Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `postgresql.enabled` | Enable PostgreSQL | `true` |
| `postgresql.image.repository` | PostgreSQL image repository | `postgres` |
| `postgresql.image.tag` | PostgreSQL image tag | `16-alpine` |
| `postgresql.auth.database` | PostgreSQL database name | `painchain` |
| `postgresql.auth.username` | PostgreSQL username | `painchain` |
| `postgresql.auth.password` | PostgreSQL password | `changeme` |
| `postgresql.primary.persistence.enabled` | Enable persistence | `true` |
| `postgresql.primary.persistence.size` | PVC size | `8Gi` |

### Redis Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `redis.enabled` | Enable Redis | `true` |
| `redis.image.repository` | Redis image repository | `redis` |
| `redis.image.tag` | Redis image tag | `7-alpine` |

### API Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `api.enabled` | Enable API service | `true` |
| `api.replicaCount` | Number of API replicas | `1` |
| `api.image.repository` | API image repository | `ghcr.io/painchain/painchain-api` |
| `api.image.tag` | API image tag | `latest` |
| `api.service.type` | Kubernetes service type | `ClusterIP` |
| `api.service.port` | Service port | `8000` |

### Frontend Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `frontend.enabled` | Enable Frontend | `true` |
| `frontend.replicaCount` | Number of Frontend replicas | `1` |
| `frontend.image.repository` | Frontend image repository | `ghcr.io/painchain/painchain-frontend` |
| `frontend.image.tag` | Frontend image tag | `latest` |
| `frontend.service.type` | Kubernetes service type | `ClusterIP` |
| `frontend.service.port` | Service port | `5173` |

### Celery Worker Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `celeryWorker.enabled` | Enable Celery Worker | `true` |
| `celeryWorker.replicaCount` | Number of worker replicas | `1` |

### Celery Beat Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `celeryBeat.enabled` | Enable Celery Beat scheduler | `true` |
| `celeryBeat.replicaCount` | Number of beat replicas | `1` |

## Accessing PainChain

After installation, follow the notes printed by Helm to access your PainChain instance.

For local development with port forwarding:

```bash
# Forward frontend
kubectl port-forward svc/painchain-frontend 5173:5173

# Forward API
kubectl port-forward svc/painchain-api 8000:8000

# Access the application
open http://localhost:5173
```

## Upgrading

```bash
helm upgrade painchain . -f custom-values.yaml
```

## Uninstalling

```bash
helm uninstall painchain
```

This will remove all resources associated with the chart, except for PersistentVolumeClaims (to prevent data loss).

To also remove PVCs:

```bash
kubectl delete pvc -l app.kubernetes.io/instance=painchain
```

## Architecture

This chart deploys the following components:

- **PostgreSQL**: Database for storing change events and configuration
- **Redis**: Message broker for Celery task queue
- **API**: FastAPI backend service
- **Celery Worker**: Background task processor for connector syncing
- **Celery Beat**: Scheduler for periodic tasks
- **Frontend**: React-based web interface

## Production Recommendations

1. **Use a strong database password**:
   ```bash
   --set postgresql.auth.password=your-secure-password
   ```

2. **Configure resource limits**:
   Edit `values.yaml` to set appropriate resource requests and limits for your workload.

3. **Enable ingress**:
   Configure ingress in `values.yaml` to expose the application externally.

4. **Use external database** (optional):
   For production, consider using a managed database service and disable the built-in PostgreSQL.

5. **Configure persistent storage**:
   Ensure your cluster has a storage class that supports ReadWriteOnce access mode.

## License

Apache License 2.0
