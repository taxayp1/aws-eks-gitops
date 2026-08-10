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

CodePipeline builds + scans image ──► pushes to ECR ──► bumps tag in this repo
│
ArgoCD (watching this repo) ──► deploys to EKS
│
External Secrets Operator ──► AWS Secrets Manager ──► K8s secrets ──► pods
│
app pods ──► RDS PostgreSQL (SSL)


## Design notes

- **No plaintext secrets** — manifests reference secret *names* in AWS Secrets Manager
  via External Secrets Operator; no credential values are committed here.
- **ClusterSecretStore** (not namespaced SecretStore) — required so the store can
  reference the ESO service account in a different namespace via IRSA.
- **RDS SSL** — the app connects to RDS with SSL enforced (`NODE_TLS_REJECT_UNAUTHORIZED`
  handling for the RDS CA), which the homelab's self-managed PostgreSQL did not require.

## Related

- **Infrastructure (Terraform):** [aws-eks-terraform-infra](https://github.com/taxayp1/aws-eks-terraform-infra)