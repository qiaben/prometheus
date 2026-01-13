# Prometheus Deployment

Prometheus monitoring and alerting system deployed using Kustomize.

## Structure

```
prometheus/
├── base/                    # Base Prometheus configuration
│   ├── namespace.yaml       # monitoring namespace
│   ├── rbac.yaml           # ServiceAccount, ClusterRole, ClusterRoleBinding
│   ├── configmap.yaml      # Prometheus configuration
│   ├── pvc.yaml            # Persistent storage (50Gi)
│   ├── deployment.yaml     # Prometheus deployment
│   ├── service.yaml        # ClusterIP service
│   ├── ingress.yaml        # Ingress configuration
│   └── kustomization.yaml  # Base kustomization
├── overlays/
│   ├── stage/              # Stage environment
│   │   └── kustomization.yaml
│   └── prod/               # Production environment
│       └── kustomization.yaml
└── README.md
```

## Environments

### Stage
- **URL:** https://prometheus.apps-stage.in.hinisoft.com
- **Ingress:** nginx-private (internal only, via VPN)
- **Replicas:** 1
- **Resources:** 500m CPU / 1Gi Memory
- **Storage:** 50Gi
- **Retention:** 30 days

### Production
- **URL:** https://prometheus.apps-prod.in.hinisoft.com
- **Ingress:** nginx-public (external access)
- **Replicas:** 2
- **Resources:** 1000m CPU / 2Gi Memory
- **Storage:** 100Gi
- **Retention:** 90 days

## Deployment

### Stage Environment
```bash
kubectl apply -k overlays/stage
```

### Production Environment
```bash
kubectl apply -k overlays/prod
```

## Features

- Prometheus v3.1.0
- Persistent storage with Longhorn
- Automatic SSL certificates via cert-manager + Let's Encrypt
- Kubernetes service discovery (pods, services, nodes, endpoints)
- Health checks (liveness and readiness probes)
- Resource limits and requests
- RBAC with ServiceAccount and ClusterRole

## Scrape Configurations

Prometheus is configured to automatically discover and scrape:
- Kubernetes API servers
- Kubernetes nodes
- Kubernetes pods (with `prometheus.io/scrape: "true"` annotation)
- Kubernetes service endpoints (with `prometheus.io/scrape: "true"` annotation)
- cAdvisor metrics

## Storage

Uses Longhorn StorageClass for persistent data storage at `/prometheus`.

## SSL Certificates

Certificates are automatically issued by cert-manager using Let's Encrypt with GoDaddy DNS-01 challenge.

## Accessing Prometheus

### Stage (Internal)
1. Connect to VPN
2. Access: https://prometheus.apps-stage.in.hinisoft.com

### Production (External)
Access directly: https://prometheus.apps-prod.in.hinisoft.com

## Integration with Grafana

Prometheus can be added as a data source in Grafana:
- **URL:** http://stage-prometheus.monitoring.svc.cluster.local:9090 (for stage)
- **Access:** Server (default)
- **Auth:** None required (internal cluster access)
