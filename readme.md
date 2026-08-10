# AWS EKS GitOps — SportsOdds Manifests

Kubernetes manifests for the SportsOdds workload running on AWS EKS, reconciled by
ArgoCD. This is the GitOps repo for the cloud environment — the AWS counterpart to a
self-hosted Kubernetes homelab running the same workload.

ArgoCD watches this repo and deploys any change automatically. The CI pipeline
(AWS CodePipeline + CodeBuild) bumps the image tag here after building and scanning,
which triggers a deployment — no manual `kubectl apply`.

## What's here

| File | Purpose |
|------|---------|
| `namespace.yaml` | The `sportsodds` namespace |
| `sportsodd.yaml` | Deployment + Service for the web app |
| `sports-cronfetch.yaml` | CronJob — scheduled odds fetch |
| `racing-cronfetch.yaml` | CronJob — racing odds (5-min refresh during racing hours) |
| `secretstore.yaml` | ClusterSecretStore — connects External Secrets Operator to AWS Secrets Manager |
| `external-secret-rds.yaml` | Pulls RDS credentials from Secrets Manager into a K8s secret |
| `external-secret-app.yaml` | Pulls application API keys from Secrets Manager |

## How it fits together