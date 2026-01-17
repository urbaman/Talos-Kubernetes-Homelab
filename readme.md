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
    |   ├── cilium
    |   |   ├── cilium-app-argocd.yaml
    |   |   ├── cilium-values.yaml
    |   |   └── argoCD-pre-deploy
    |   |       └── argoCD-pre-deploy-manifest.yaml
    |   └── metricsServer
    |       ├── kubeletServingCertApprover-manifest.yaml #copy the kubelet-serving-cert-approver manifest, standalone or high-availability
    |       └── metricsServer-manifest.yaml #copy the metrics server manifest, standalone or high-availability
    ├── pre-production # pre-production setup
    |   ├── externalSecretsOperator
    |   |   ├── externalSecretsOperator-app-argocd.yaml
    |   |   └── externalSecretsOperator-post-deploy
    |   |       └── externalSecretsOperator-ExternalSecretStore-manifest.yaml
    |   ├── localPathProvisioner
    |   |   └── localPathProvisioner-manifest.yaml
    |   ├── sealedSecrets
    |   ├── rookCeph
    |   |   ├── rookCeph-app-argocd.yaml
    |   |   ├── rookCephCluster-app-argocd.yaml
    |   |   └── rookCephCluster-values.yaml
    |   ├── csiDriverNfs
    |   |   ├── csiDriverNfs-app-argocd.yaml
    |   |   ├── csiDriverNfs-values.yaml
    |   |   └── csiDriverNfs-post-deploy
    |   |       └── csiDriverNfs-storageClass-volumeSnapshotClass-manifest.yaml
    |   ├── valkey
    |   |   ├── valkey-pre-deploy
    |   |   |   └── valkey-pre-deploy-manifest.yaml
    |   |   ├── valkey-app-argocd.yaml
    |   |   └── valkey-values.yaml
    |   └── argoCD
    |   |   ├── argoCD-pre-deploy
    |   |   |   └── argoCD-pre-deploy-manifest.yaml
    |   |   ├── argoCD-app-argocd.yaml
    |   |   └── argoCD-values.yaml
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
