# Strategic Capacity Planning & Forecasting Workshop for OpenShift

A comprehensive full-day workshop teaching platform engineers, SREs, and developers how to transition from reactive "firefighting" to predictive, data-driven capacity management using Red Hat OpenShift.

## Overview

This workshop uses hands-on simulation to teach 7 core competencies:

1. **Planning Horizon & Baselines** - Tactical vs Strategic planning, current state audits
2. **Mathematics of Forecasting** - Pod Velocity, PromQL, ACM forecasting dashboards
3. **Developer Track** - QoS classes, CPU throttling, resource right-sizing, HPA
4. **Infrastructure Track** - Fleet sizing, node density optimization, etcd limits
5. **Fleet Observability** - RHACM multi-cluster dashboards, custom metrics
6. **Integration Challenge** - Live simulation: product launch spike + AZ failure
7. **Strategic Roadmapping** - Building 12-month capacity plans

## Architecture

This workshop is deployed as a multi-component Helm chart using the **Field-Sourced Content** pattern with **ArgoCD App-of-Apps** architecture.

### Components

1. **RHACM** (`components/rhacm/`) - Red Hat Advanced Cluster Management
   - Operator subscription (release-2.14)
   - MultiClusterHub with observability enabled
   - Thanos metrics storage (ODF backend)
   - Custom capacity metrics allowlist

2. **Monitoring Config** (`components/monitoring-config/`) - PromQL & Alerting
   - 27 PromQL queries for all 7 workshop modules
   - Prometheus alerting rules

3. **Sample Apps** (`components/sample-apps/`) - Workshop Applications
   - QoS Demos: BestEffort, Burstable, Guaranteed pods
   - Load Generator, Noisy Neighbor, Critical App, Batch Job
   - HPA, ResourceQuota, LimitRange, PriorityClasses

4. **Simulation Toolkit** (`components/simulation-toolkit/`) - Module 6 Chaos Game
   - RBAC for node operations
   - Traffic spike job, simulation scripts

5. **Node Density Config** (`components/node-density-config/`) - Module 4 Tuning
   - KubeletConfig for maxPods tuning (disabled by default)

6. **Showroom** (`components/showroom/`) - Lab Guide UI
   - Antora-based lab guide with integrated terminal

## Prerequisites

- OpenShift 4.14+ cluster
- OpenShift GitOps (ArgoCD) operator installed
- Storage class for RHACM observability (ODF Ceph RBD recommended)
- Cluster admin privileges
- ~20 CPU cores and ~40GB RAM available

## Quick Start

### Deploy via ArgoCD (Recommended)

```bash
CLUSTER_DOMAIN="apps.cluster-mks77.dynamic.redhatworkshops.io"
GUID="workshop01"

cat <<EOF | oc apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: capacity-planning-workshop
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://github.com/tosin2013/openshift-capacity-planning-workshop.git
    targetRevision: main
    path: .
    helm:
      values: |
        global:
          namespace: capacity-workshop
          clusterDomain: "${CLUSTER_DOMAIN}"
          guid: "${GUID}"
        deployer:
          domain: "${CLUSTER_DOMAIN}"
        components:
          rhacm:
            enabled: true
          monitoringConfig:
            enabled: true
          sampleApps:
            enabled: true
          simulationToolkit:
            enabled: true
          nodeDensityConfig:
            enabled: false
          showroom:
            enabled: true
  destination:
    server: https://kubernetes.default.svc
    namespace: capacity-workshop
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF
```

### Deploy via Helm CLI

```bash
git clone https://github.com/tosin2013/openshift-capacity-planning-workshop.git
cd openshift-capacity-planning-workshop

helm install capacity-workshop . \
  --namespace capacity-workshop \
  --create-namespace \
  --set global.clusterDomain="apps.cluster-mks77.dynamic.redhatworkshops.io" \
  --set global.guid="workshop01" \
  --set deployer.domain="apps.cluster-mks77.dynamic.redhatworkshops.io" \
  --wait --timeout=30m
```

## Post-Deployment Verification

```bash
# Check RHACM
oc get mch -n open-cluster-management -o jsonpath='{.items[0].status.phase}'
# Expected: Running

# Check sample apps
oc get pods -n capacity-workshop

# Get Showroom URL
oc get route -n capacity-workshop showroom -o jsonpath='https://{.spec.host}'
```

## Troubleshooting

See complete troubleshooting guide in the repository documentation.

## Related Repositories

- **Lab Guide Content**: https://github.com/tosin2013/capacity-planning-lab-guide
- **AgnosticD Workload**: https://github.com/tosin2013/agnosticd-v2

## Maintainer

Tosin Akinosho (takinosho@redhat.com)

## License

Apache License 2.0
