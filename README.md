# DevOps EduCeny Helm Charts

Official Helm charts repository for the DevOps EduCeny project infrastructure.

## Available Charts

### 🚀 nestjs-app

Production-ready Helm chart for NestJS/Next.js applications with enterprise-grade features:

- **Auto-scaling**: HorizontalPodAutoscaler with CPU/Memory metrics
- **Security**: External Secrets integration with AWS Secrets Manager
- **Networking**: Ingress with NGINX controller support
- **Health Checks**: Kubernetes liveness and readiness probes
- **Cost Optimization**: Resource limits and Karpenter integration
- **Observability**: Structured logging and monitoring ready

## 📦 Installation

### Add Helm Repository

```bash
helm repo add educeny https://mhm0ud.github.io/devops-educeny-helm-charts/
helm repo update
```

### Install Chart

```bash
helm install my-app educeny/nestjs-app \
  --namespace dev \
  --create-namespace \
  --values values.yaml
```

## 🛠️ Chart Configuration

### Basic Configuration

```yaml
# values.yaml
app:
  name: "my-app"
  environment: "dev"

image:
  repository: "547049679514.dkr.ecr.eu-north-1.amazonaws.com/my-app"
  tag: "latest"

ingress:
  enabled: true
  hosts:
    - host: my-app.dev.example.com
      paths:
        - path: /
          pathType: Prefix
```

### Environment-Specific Configuration

#### Development Environment

```yaml
# dev-values.yaml
tolerations:
  - key: workload-type
    value: dev
    effect: NoSchedule

resources:
  requests:
    cpu: 100m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  minReplicas: 1
  maxReplicas: 3

configMaps:
  data:
    NATS_URL: "nats://nats-dev.dev.svc.cluster.local:4222"
    NODE_ENV: "development"
    LOG_LEVEL: "debug"

externalSecrets:
  enabled: true
  secretStore: "aws-secrets-manager-dev"
  secrets:
    - secretName: "dev-database-credentials"
      secretKey: "password"
      envName: "DB_PASSWORD"
```

#### Production Environment

```yaml
# prod-values.yaml
replicaCount: 2

tolerations:
  - key: workload-type
    value: prod
    effect: NoSchedule

resources:
  requests:
    cpu: 200m
    memory: 512Mi
  limits:
    cpu: 1000m
    memory: 1Gi

autoscaling:
  minReplicas: 2
  maxReplicas: 5

configMaps:
  data:
    NATS_URL: "nats://nats-prod.prod.svc.cluster.local:4222"
    NODE_ENV: "production"
    LOG_LEVEL: "info"

externalSecrets:
  enabled: true
  secretStore: "aws-secrets-manager-prod"
  secrets:
    - secretName: "prod-database-credentials"
      secretKey: "password"
      envName: "DB_PASSWORD"
```

## 🏗️ Architecture Integration

This chart is designed to work with:

- **ArgoCD**: GitOps-based deployment automation
- **Karpenter**: Cost-optimized node auto-scaling
- **External Secrets Operator**: Secure secret management
- **NGINX Ingress Controller**: Load balancing and routing
- **NATS**: Inter-service messaging

## 📋 Prerequisites

- Kubernetes 1.24+
- Helm 3.8+
- External Secrets Operator installed
- NGINX Ingress Controller installed
- AWS Secrets Manager configured (for External Secrets)

## 🔧 Development

### Testing Charts Locally

```bash
# Clone repository
git clone https://github.com/mhm0ud/devops-educeny-helm-charts.git
cd devops-educeny-helm-charts

# Lint chart
helm lint nestjs-app/

# Dry run installation
helm install test-release nestjs-app/ \
  --dry-run \
  --debug \
  --values test-values.yaml

# Template generation
helm template test-release nestjs-app/ \
  --values test-values.yaml \
  --output-dir ./output
```

### Chart Structure

```
nestjs-app/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Default configuration
└── templates/
    ├── deployment.yaml     # Kubernetes Deployment
    ├── service.yaml        # Kubernetes Service
    ├── ingress.yaml        # Ingress configuration
    ├── configmap.yaml      # ConfigMap for app config
    ├── externalsecret.yaml # External Secrets integration
    ├── serviceaccount.yaml # Service Account + RBAC
    ├── hpa.yaml           # Horizontal Pod Autoscaler
    └── _helpers.tpl       # Helm template helpers
```

## 🚀 CI/CD Integration

### GitHub Actions Example

```yaml
- name: Deploy with Helm
  run: |
    helm upgrade --install ${{ env.APP_NAME }} \
      educeny/nestjs-app \
      --namespace dev \
      --create-namespace \
      --values .helm/dev-values.yaml \
      --set image.tag=${{ github.sha }}
```

### ArgoCD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-dev
spec:
  source:
    repoURL: https://mhm0ud.github.io/devops-educeny-helm-charts/
    chart: nestjs-app
    targetRevision: 0.1.0
    helm:
      valueFiles:
        - values-dev.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
```

## 📊 Monitoring & Observability

The chart includes built-in support for:

- **Health Endpoints**: `/health` and `/ready` for Kubernetes probes
- **Metrics Collection**: Prometheus-compatible metrics endpoint
- **Structured Logging**: JSON logs with correlation IDs
- **Distributed Tracing**: OpenTelemetry ready

## 💰 Cost Optimization Features

- **Resource Limits**: Prevent resource over-allocation
- **HPA Configuration**: Auto-scale based on actual usage
- **Karpenter Integration**: Automatic node provisioning/de-provisioning
- **Spot Instance Support**: Tolerations for cost-optimized nodes

## 📝 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes and test thoroughly
4. Update chart version in `Chart.yaml`
5. Commit your changes (`git commit -m 'Add amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

## 📞 Support

For issues and questions:
- Create an issue on GitHub
- Check existing documentation
- Review chart values and templates