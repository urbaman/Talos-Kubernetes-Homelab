# Creating the Cluster

## Prerequisites

- Git Server (Gitea)
- Secrets Vault (OpenBao, for External Secrets operator)

**Design notes:**

- We will use hostpath for database clusters, setting three nodes for scheduling the clusters. You need to install a proper database-ready shared storage to replace this (rookCeph?) but in a homeLab environment having both database cluster replicas and storage replicas would be too much of a bottleneck. Rook Ceph is there just for an example. You can also use NFS, but be aware of the settings to make it database-ready, as some of them do not support NFS or need specific settings.
- ...

```markdown
Talos-Kubernetes-Homelab
└── cluster
    ├── bootstrap
    |   ├── control-panel.yaml
    |   ├── hostname.yaml
    |   ├── worker.yaml
    |   ├── worker-database.yaml
    |   ├── worker-gpu.yaml
    |   └── cilium
    |       ├── cilium-app-argocd.yaml
    |       ├── cilium-values.yaml
    |       └── cilium-post-deploy
    |           └── cilium-lb-manifest.yaml
    ├── pre-production # pre-production setup
    |   ├── automation
    |   |   ├── argoCD
    |   |   |   ├── argoCD-pre-deploy
    |   |   |   |   └── argoCD-pre-deploy-manifest.yaml
    |   |   |   ├── argoCD-app-argocd.yaml
    |   |   |   └── argoCD-values.yaml
    |   |   ├── certManager
    |   |   |   ├── certManager-app-argocd.yaml
    |   |   |   └── certManager-values.yaml
    |   |   ├── externalSecretsOperator
    |   |   |   ├── externalSecretsOperator-pre-deploy-manual
    |   |   |   |   └── externalSecretsOperator-ExternalSecretStore-secret.yaml
    |   |   |   ├── externalSecretsOperator-post-deploy
    |   |   |   |   └── externalSecretsOperator-ExternalSecretStore-manifest.yaml
    |   |   |   └── externalSecretsOperator-app-argocd.yaml
    |   |   └── automation-apps-argocd.yaml
    |   ├── databases
    |   |   ├── valkey
    |   |   |   ├── valkey-pre-deploy
    |   |   |   |   └── valkey-pre-deploy-manifest.yaml
    |   |   |   ├── valkey-app-argocd.yaml
    |   |   |   └── valkey-values.yaml
    |   |   └── databases-apps-argocd.yaml
    |   ├── observability
    |   |   ├── metricsServer
    |   |   |   ├── kubeletServingCertApprover-app-argocd.yaml
    |   |   |   ├── kubeletServingCertApprover-manifest.yaml
    |   |   |   ├── metricsServer-app-argocd.yaml
    |   |   |   ├── metricsServer-values.yaml
    |   |   |   └── metricsServer-manifest.yaml #copy the metrics server manifest, standalone or high-availability
    |   |   └── observability-apps-argocd.yaml
    |   └── storage
    |       ├── csiDriverNfs
    |       |   ├── csiDriverNfs-app-argocd.yaml
    |       |   ├── csiDriverNfs-values.yaml
    |       |   └── csiDriverNfs-post-deploy
    |       |       └── csiDriverNfs-storageClass-volumeSnapshotClass-manifest.yaml
    |       ├── localPathProvisioner
    |       |   ├── localPathProvisioner-app-argocd.yaml
    |       |   └── localPathProvisioner-manifest.yaml
    |       ├── rookCeph
    |       |   ├── rookCeph-app-argocd.yaml
    |       |   ├── rookCephCluster-app-argocd.yaml
    |       |   └── rookCephCluster-values.yaml
    |       └── databases-apps-argocd.yaml
    └── production # production setup
```

## Basic bootstrapping

**Beware:** use helm template instead of helm install or upgrade -i when installing things like Cilium or other Pre-Production stuff if you're going to use ArgoCD for GitOps, as it makes use of helm template, and you'll make the tools manageable through ArgoCD

```bash
helm template cilium cilium/cilium -n kube-system -f cilium-values.yaml > cilium.yaml
kubectl apply -f cilium.yaml
```

[Deploy, with or without Omni](https://github.com/urbaman/HomeLab/tree/88eea8da4458b10f44be4f9132422f3b08641766/Kubernetes/Cluster/05-Talos)

## Pre-production

Manually install basic tools, without service/pod monitors, through manifests, eventually templating helm apps (doable with ansible) if using ArgoCD for GitOps.

- Metrics Server
- External Secrets Operator (with OpenBao)
- Sealed Secrets (with Git)
- Storage (for Valkey)
- Valkey (for ArgoCD)
- ArgoCD

## Production

Point ESO to the Secrets Vault or SS to the Git Server and get the secrets, then point ArgoCD to a repo with the proper ArgoApps to (re)deploy

- Storage (NFS, Ceph,... if not already deployed for Valkey)
- Monitoring (Prometheus Stack, and service/pod monitors where not already deployed: CNI, ArgoCD, ESO/SS, Storage, ...)
- Cert-manager
- Ingress (Traefik)
- Metallb (if needed)
- Databases (other than Valkey)
- ...
