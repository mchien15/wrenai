# WrenAI - Deployment Guide

## Overview

This guide covers deploying WrenAI in different environments: local development (Docker Compose) and production (Kubernetes).

## Prerequisites

### Local Development
- Docker Desktop 20.10+
- Docker Compose 2.0+
- 8GB RAM minimum (16GB recommended)
- 20GB disk space

### Production
- Kubernetes 1.25+ (or managed K8s service)
- kubectl configured
- Helm 3.x (optional, for Helm charts)
- Ingress controller (NGINX, Traefik, etc.)
- PostgreSQL 14+ database
- Object storage (optional, for backups)

## Local Development Deployment

### Quick Start

1. **Clone Repository**:
```bash
git clone https://github.com/Canner/WrenAI.git
cd WrenAI
```

2. **Configure Environment**:
```bash
cd docker
cp .env.example .env.local
cp config.example.yaml config.yaml
```

3. **Edit `.env.local`**:
```bash
# Required: LLM Provider API Key
OPENAI_API_KEY=sk-your-openai-key-here

# Optional: Other LLM providers
ANTHROPIC_API_KEY=your-anthropic-key
GOOGLE_API_KEY=your-google-key

# Service Ports (defaults shown)
WREN_UI_PORT=3000
WREN_AI_SERVICE_PORT=5556
WREN_ENGINE_PORT=8080
IBIS_SERVER_PORT=8000
QDRANT_PORT=6333
```

4. **Edit `config.yaml`** (AI Service Configuration):
```yaml
llm:
  provider: openai
  model: gpt-4-turbo-preview
  api_key_env: OPENAI_API_KEY
  temperature: 0.0
  max_tokens: 2000

embedder:
  provider: openai
  model: text-embedding-3-small
  api_key_env: OPENAI_API_KEY

document_store:
  provider: qdrant
  url: http://qdrant:6333
  collection_name: wrenai

engine:
  url: http://wren-engine:8080
```

5. **Start Services**:
```bash
docker compose --env-file .env.local up -d
```

6. **Verify Deployment**:
```bash
# Check all services are running
docker compose ps

# View logs
docker compose logs -f

# Check health
curl http://localhost:3000/health
curl http://localhost:5556/health
```

7. **Access UI**:
```
http://localhost:3000
```

### Service Overview

| Service | Port | Purpose | Health Check |
|---------|------|---------|--------------|
| wren-ui | 3000 | Frontend + GraphQL | `GET /health` |
| wren-ai-service | 5556 | AI/LLM service | `GET /health` |
| wren-engine | 8080 | SQL engine | `GET /health` |
| ibis-server | 8000 | SQL abstraction | N/A |
| qdrant | 6333 | Vector DB | `GET /` |
| bootstrap | N/A | Initialization | One-time setup |

### Development Workflow

#### Start Services
```bash
docker compose --env-file .env.local up -d
```

#### Stop Services
```bash
docker compose down
```

#### View Logs
```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f wren-ai-service
```

#### Restart Service
```bash
docker compose restart wren-ai-service
```

#### Rebuild Service (after code changes)
```bash
docker compose up -d --build wren-ui
```

#### Reset Database
```bash
docker compose down -v
docker compose --env-file .env.local up -d
```

### Troubleshooting

**Issue**: Services fail to start
```bash
# Check port conflicts
lsof -i :3000
lsof -i :5556

# Check Docker resources
docker info
```

**Issue**: AI Service returns errors
```bash
# Verify API key
echo $OPENAI_API_KEY

# Check AI Service logs
docker compose logs wren-ai-service
```

**Issue**: Qdrant connection errors
```bash
# Restart Qdrant
docker compose restart qdrant

# Clear Qdrant data
docker compose down -v
docker compose --env-file .env.local up -d
```

## Production Deployment (Kubernetes)

### Architecture Overview

```
Internet
  ↓
Ingress Controller (NGINX/Traefik)
  ↓
┌─────────────────────────────────────┐
│         Kubernetes Cluster          │
│  ┌──────────────┐  ┌──────────────┐ │
│  │ wren-ui      │  │ wren-ai      │ │
│  │ Deployment   │  │ service      │ │
│  └──────────────┘  └──────────────┘ │
│  ┌──────────────┐  ┌──────────────┐ │
│  │ wren-engine  │  │ ibis-server  │ │
│  └──────────────┘  └──────────────┘ │
│  ┌──────────────┐                   │
│  │ qdrant       │                   │
│  │ StatefulSet  │                   │
│  └──────────────┘                   │
└─────────────────────────────────────┘
  ↓
External Services
  - PostgreSQL (managed)
  - Object Storage (optional)
```

### Deployment Steps

#### 1. Prepare Kubernetes Cluster

**Option A**: Cloud-managed K8s
- Google GKE
- Amazon EKS
- Azure AKS
- DigitalOcean K8s

**Option B**: Self-hosted
- kubeadm
- Rancher
- MicroK8s

#### 2. Install Ingress Controller

**NGINX Ingress**:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```

**Traefik Ingress**:
```bash
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v2.10/docs/content/reference/dynamic-configuration/kubernetes-crd-definition-v1.yml
```

#### 3. Deploy PostgreSQL

**Option A**: Managed PostgreSQL (Recommended)
- Cloud SQL (GCP)
- RDS (AWS)
- Azure Database for PostgreSQL

**Option B**: Self-hosted with Helm
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install wren-postgres bitnami/postgresql \
  --set auth.postgresPassword=your-password \
  --set auth.database=wrenai \
  --set persistence.size=20Gi
```

#### 4. Create Namespace and Secrets

```bash
# Create namespace
kubectl create namespace wrenai

# Create secrets for API keys
kubectl create secret generic wrenai-secrets \
  --from-literal=OPENAI_API_KEY=sk-your-key \
  --from-literal=POSTGRES_URL=postgresql://user:pass@host:5432/wrenai \
  -n wrenai
```

#### 5. Deploy Qdrant StatefulSet

```bash
kubectl apply -f deployment/base/qdrant-statefulset.yaml -n wrenai
```

#### 6. Deploy Applications

```bash
# Deploy wren-engine
kubectl apply -f deployment/base/wren-engine-deployment.yaml -n wrenai

# Deploy ibis-server
kubectl apply -f deployment/base/ibis-server-deployment.yaml -n wrenai

# Deploy wren-ai-service
kubectl apply -f deployment/base/wren-ai-service-deployment.yaml -n wrenai

# Deploy wren-ui
kubectl apply -f deployment/base/wren-ui-deployment.yaml -n wrenai
```

#### 7. Configure Ingress

```bash
kubectl apply -f deployment/base/ingress.yaml -n wrenai
```

#### 8. Verify Deployment

```bash
# Check pod status
kubectl get pods -n wrenai

# Check services
kubectl get svc -n wrenai

# Check ingress
kubectl get ingress -n wrenai

# View logs
kubectl logs -f deployment/wren-ai-service -n wrenai
```

### Configuration Management

#### Kustomize Overlays

```bash
# Base configuration
kubectl apply -k deployment/base

# Production overlay
kubectl apply -k deployment/overlays/production

# Staging overlay
kubectl apply -k deployment/overlays/staging
```

#### Helm Charts (Optional)

```bash
# Create values file
cat > values.yaml <<EOF
replicaCount: 3

image:
  repository: ghcr.io/canner/wrenai-ui
  tag: "v1.0.0"

env:
  - name: POSTGRES_URL
    valueFrom:
      secretKeyRef:
        name: wrenai-secrets
        key: POSTGRES_URL

ingress:
  enabled: true
  host: wrenai.example.com
EOF

# Deploy
helm install wrenai ./deployment/charts/wrenai -f values.yaml -n wrenai
```

### Scaling

#### Horizontal Pod Autoscaler

```bash
# Auto-scale wren-ui
kubectl autoscale deployment wren-ui \
  --cpu-percent=70 \
  --min=3 \
  --max=10 \
  -n wrenai

# Auto-scale wren-ai-service
kubectl autoscale deployment wren-ai-service \
  --cpu-percent=70 \
  --min=3 \
  --max=10 \
  -n wrenai
```

#### Manual Scaling

```bash
# Scale to 5 replicas
kubectl scale deployment wren-ui --replicas=5 -n wrenai
```

### Monitoring

#### Health Checks

```bash
# Liveness probe
kubectl get pods -n wrenai -o jsonpath='{.items[*].status.containerStatuses[*].state.running}'

# Service health
curl http://wrenai.example.com/health
```

#### Logging

```bash
# Stream logs
kubectl logs -f deployment/wren-ai-service -n wrenai

# All pods
kubectl logs -f -l app=wrenai -n wrenai

# Previous instance
kubectl logs --previous deployment/wren-ai-service -n wrenai
```

#### Metrics (Prometheus + Grafana)

```bash
# Install Prometheus Operator
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring

# Access Grafana
kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring
```

### Backup and Restore

#### Database Backup

```bash
# Backup
kubectl exec -it wren-postgres-0 -n wrenai -- pg_dump wrenai > backup.sql

# Restore
cat backup.sql | kubectl exec -i wren-postgres-0 -n wrenai -- psql wrenai
```

#### Qdrant Backup

```bash
# Backup Qdrant data
kubectl exec -it qdrant-0 -n wrenai -- tar czf /tmp/qdrant-backup.tar.gz /qdrant/storage
kubectl cp wrenai/qdrant-0:/tmp/qdrant-backup.tar.gz ./qdrant-backup.tar.gz
```

## Environment Variables Reference

### wren-ui

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DB_TYPE` | No | `sqlite` | Database type (sqlite, pg) |
| `POSTGRES_URL` | Conditional | - | PostgreSQL connection string |
| `WREN_ENGINE_ENDPOINT` | Yes | `http://wren-engine:8080` | Wren Engine URL |
| `WREN_AI_ENDPOINT` | Yes | `http://wren-ai-service:5556` | AI Service URL |
| `IBIS_SERVER_ENDPOINT` | Yes | `http://ibis-server:8000` | Ibis Server URL |

### wren-ai-service

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `OPENAI_API_KEY` | Conditional | - | OpenAI API key |
| `ANTHROPIC_API_KEY` | No | - | Anthropic API key |
| `GOOGLE_API_KEY` | No | - | Google API key |
| `QDRANT_URL` | No | `http://localhost:6333` | Qdrant URL |
| `QDRANT_API_KEY` | No | - | Qdrant API key |

### Common

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `TZ` | No | `UTC` | Timezone |
| `LOG_LEVEL` | No | `info` | Logging level |
| `NODE_ENV` | No | `production` | Environment mode |

## Security Best Practices

### Secrets Management

```bash
# Use Kubernetes secrets (not in git)
kubectl create secret generic wrenai-secrets \
  --from-env-file=.env.production \
  -n wrenai

# Encrypt secrets with Sealed Secrets
kubeseal -f wrenai-secrets.yaml -w sealed-secrets.yaml

# Use external secrets (AWS Secrets Manager, Vault)
```

### Network Policies

```bash
# Restrict pod-to-pod communication
kubectl apply -f deployment/base/network-policy.yaml -n wrenai
```

### TLS/SSL

```bash
# Cert-Manager for automatic certificates
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Create ClusterIssuer
kubectl apply -f deployment/base/cert-manager-issuer.yaml
```

### RBAC

```bash
# Create service accounts
kubectl create serviceaccount wrenai -n wrenai

# Create roles
kubectl apply -f deployment/base/rbac.yaml
```

## Performance Tuning

### Database Optimization

```sql
-- Add indexes
CREATE INDEX idx_questions_project_id ON questions(project_id);
CREATE INDEX idx_models_data_source_id ON models(data_source_id);

-- Connection pooling (PgBouncer)
-- Recommended: 25 connections per instance
```

### AI Service Optimization

```yaml
# config.yaml
llm:
  max_tokens: 2000
  temperature: 0.0
  request_timeout: 30

pipeline:
  max_sql_correction_retries: 3
  enable_caching: true
  cache_ttl: 3600
```

### Resource Limits

```yaml
# deployment.yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "2Gi"
    cpu: "2000m"
```

## Troubleshooting Production

### Pod Not Starting

```bash
# Describe pod
kubectl describe pod wren-ui-xxx -n wrenai

# Check events
kubectl get events -n wrenai --sort-by='.lastTimestamp'

# Check logs
kubectl logs wren-ui-xxx -n wrenai
```

### Database Connection Issues

```bash
# Test connection from pod
kubectl exec -it wren-ui-xxx -n wrenai -- nc -zv postgres 5432

# Check database logs
kubectl logs wren-postgres-0 -n wrenai
```

### High Memory Usage

```bash
# Check resource usage
kubectl top pods -n wrenai

# Increase limits
kubectl set resources deployment wren-ui \
  --limits=memory=4Gi,cpu=4000m \
  -n wrenai
```

### Slow Queries

```bash
# Check AI Service logs for slow LLM calls
kubectl logs deployment/wren-ai-service -n wrenai | grep "duration"

# Enable query logging
# config.yaml: logging.level: DEBUG
```

## Upgrades

### Rolling Update

```bash
# Update image
kubectl set image deployment/wren-ui \
  wren-ui=ghcr.io/canner/wrenai-ui:v1.1.0 \
  -n wrenai

# Watch rollout status
kubectl rollout status deployment/wren-ui -n wrenai
```

### Rollback

```bash
# Rollback to previous version
kubectl rollout undo deployment/wren-ui -n wrenai

# Rollback to specific revision
kubectl rollout undo deployment/wren-ui --to-revision=2 -n wrenai
```

## Migration Guides

### SQLite to PostgreSQL

1. Export SQLite data
2. Import to PostgreSQL
3. Update `DB_TYPE=pg` and `POSTGRES_URL`
4. Restart services

### Version Upgrades

1. Read release notes
2. Backup database
3. Test in staging
4. Rolling update in production
5. Verify health checks
6. Monitor for issues

## Support

- **Documentation**: https://docs.getwren.ai/
- **GitHub Issues**: https://github.com/Canner/WrenAI/issues
- **Discord**: https://discord.gg/5DvshJqG8Z
- **Email**: support@wrenai.com

---

**Document Version**: 1.0
**Last Updated**: 2026-03-29
**Maintainer**: WrenAI Team
