Here's the end-to-end flow of how Episode 10 works:

---

## End-to-End Flow

### Phase 1: Infrastructure Setup (ONE TIME — `terraform apply`)

```
You run: GitHub Actions → ep10-setup.yml → apply

Terraform does this in order:

1. Creates VPC (10.0.0.0/16)
   └── 2 public subnets (for ALB, Bastion)
   └── 2 private subnets (for EKS worker nodes)
   └── NAT Gateway (private subnets → internet)
   └── Internet Gateway (public subnets → internet)

2. Creates EKS Cluster (Auto Mode)
   └── IAM role for cluster (6 policies)
   └── IAM role for nodes (ECR pull, Secrets Manager)
   └── KMS key (encrypts K8s secrets at rest)
   └── Security group (443 from VPC)
   └── All 5 log types → CloudWatch
   └── Auto Mode = AWS manages nodes automatically (no node groups)

3. Creates StorageClass (auto-ebs-sc)
   └── EBS gp3 volumes, encrypted
   └── Required for Prometheus PVC, Elasticsearch PVC

4. Creates Bastion EC2
   └── Amazon Linux 2023, t2.medium, 30GB
   └── IAM Admin role (for kubectl, aws-cli)
   └── SSM access (no SSH key needed)
   └── tools.sh installs: kubectl, helm, eksctl, docker, SonarQube
   └── EKS access entry → cluster admin

5. Creates 11 ECR Repositories
   └── frontend, cartservice, checkoutservice, etc.
   └── Lifecycle: untagged images deleted after 7 days
   └── Scan on push enabled

6. Installs K8s Delegate (Helm)
   └── Namespace: harness-delegate-ng
   └── 2 replicas (HA), autoscale to 6 at 70% CPU
   └── Resources: 500m-1 CPU, 2Gi-4Gi memory
   └── Tag: eks-k8s-delegate

7. Creates Delegate RBAC
   └── harness-builds namespace (for CI pods)
   └── ClusterRole (pods, deployments, services, etc.)
   └── ClusterRoleBinding → delegate ServiceAccount

8. Installs External Secrets Operator (Helm)
   └── Namespace: external-secrets
   └── ClusterSecretStore → connects to AWS Secrets Manager
   └── Creates empty AWS SM secret: online-boutique/app-secrets

9. Installs AWS Load Balancer Controller (Helm)
   └── Namespace: kube-system
   └── IAM policy for ALB/NLB management
   └── One shared ALB for all Ingress resources

10. Installs GitOps Agent (Helm)
    └── Namespace: gitops
    └── HA: controller×2, repo-server×2→5, server×2→4, redis×2
    └── Registers in Harness as ep10gitopsagent

11. Creates Harness Resources (Terraform Harness Provider)
    └── Service: online-boutique (ReleaseRepo → values.yaml)
    └── Environments: development + production
    └── Connectors: Prometheus, AWS SM, Kubernetes
    └── OPA Policy + PolicySet (On Run, severity: error)
    └── Monitored Service (Prometheus health source for CV)
    └── GitOps Repository (your GitHub repo)
    └── GitOps Cluster (in-cluster)
    └── GitOps Application (online-boutique → Episode-10/k8s/, NO auto-sync)

12. Creates ArgoCD Observability Apps (auto-sync ON)
    └── monitoring → kube-prometheus-stack Helm chart (Grafana + Prometheus)
    └── logging → Episode-10/k8s/logging/ Git manifests (EFK)
    └── jaeger → jaeger Helm chart
    └── otel-collector → Episode-10/k8s/tracing/ Git manifests
    └── Kibana Ingress (via shared ALB)

RESULT: Entire platform ready. Zero manual clicks.
```

---

### Phase 2: First Pipeline Run

```
You do: Harness → Import Pipeline → Run Pipeline

Pipeline Stage 1: CI (runs on your EKS — KubernetesDirect)
│
├── [Parallel] Security Scans:
│   ├── Gitleaks → scans source code for hardcoded secrets → JSON report
│   ├── Trivy → scans filesystem for vulnerabilities → JSON report
│   ├── OWASP → checks dependencies for known CVEs → JSON report
│   └── SonarQube → code quality + security analysis → sends to SonarQube server
│
├── [Parallel] Build & Push (3 batches of ~4 services each):
│   ├── BuildAndPushECR: frontend → ECR (OIDC connector, tag: v1)
│   ├── BuildAndPushECR: cartservice → ECR (tag: v1)
│   ├── BuildAndPushECR: checkoutservice → ECR (tag: v1)
│   ├── BuildAndPushECR: productcatalogservice → ECR (tag: v1)
│   ├── BuildAndPushECR: currencyservice → ECR (tag: v1)
│   ├── BuildAndPushECR: emailservice → ECR (tag: v1)
│   ├── BuildAndPushECR: paymentservice → ECR (tag: v1)
│   ├── BuildAndPushECR: recommendationservice → ECR (tag: v1)
│   ├── BuildAndPushECR: shippingservice → ECR (tag: v1)
│   ├── BuildAndPushECR: adservice → ECR (tag: v1)
│   └── BuildAndPushECR: loadgenerator → ECR (tag: v1)
│
│   Auth: OIDC connector (account.aws_account) — no access keys
│   Cache: Go build cache, npm cache, Gradle cache saved between runs

Pipeline Stage 2: AI Security Agent
│
└── python security_agent.py
    ├── Reads: trivy-results.json, gitleaks-report.json, owasp-report.json
    ├── Sends to OpenAI/Gemini for analysis
    └── Output: Prioritized security report + compliance status

Pipeline Stage 3: AI Deployment Risk Agent
│
└── python deployment_risk_agent.py
    ├── Checks: environment, test status, time of day, vulnerabilities found
    ├── Sends to AI for risk assessment
    └── Output: SAFE ✅ / RISKY ⚠️ / BLOCK ❌
    └── If BLOCK → pipeline fails here (exit code 1)

Pipeline Stage 4: GitOps Deploy (gitOpsEnabled: true)
│
├── Step 1: GitOpsUpdateReleaseRepo
│   ├── Creates a new branch from main
│   ├── Updates values.yaml with ALL 11 new image tags:
│   │   frontendImage: 123456.dkr.ecr.us-east-1.amazonaws.com/frontend:v1
│   │   cartserviceImage: 123456.dkr.ecr.us-east-1.amazonaws.com/cartservice:v1
│   │   ... (all 11)
│   ├── Commits the change
│   ├── Creates a Pull Request: "Deploy Online Boutique build #1"
│   └── Output: commitId (saved for rollback)
│
├── Step 2: HarnessApproval
│   ├── Pipeline PAUSES here
│   ├── You see the PR diff in Harness UI (what changed)
│   ├── You click "Approve" (or it auto-rejects after 1 day)
│   └── Once approved → continues
│
├── Step 3: MergePR
│   ├── Merges the PR into main branch
│   ├── Deletes the source branch
│   └── Now main has the new image tags
│
├── Step 4: GitOpsSync
│   ├── Tells ArgoCD: "sync NOW" (don't wait 3-min poll)
│   ├── ArgoCD reads main branch → renders Helm chart → applies to cluster
│   ├── Rolling update: new pods start → old pods terminate
│   └── Zero downtime (maxSurge: 1, maxUnavailable: 0)
│
├── Step 5: GitOpsGetAppDetails
│   ├── Queries ArgoCD: "what's the app status?"
│   └── Returns: Synced ✅ + Healthy ✅ (or Degraded ❌)
│
├── Step 6: Verify (Continuous Verification)
│   ├── Queries Prometheus for 5 minutes
│   ├── Compares: error_rate BEFORE deploy vs AFTER deploy
│   ├── Compares: request_duration (p99) BEFORE vs AFTER
│   ├── If metrics SAME or BETTER → pass ✅
│   └── If metrics WORSE → triggers rollback ❌

Notifications:
└── Slack: "Pipeline ep10-enterprise-gitops #1 succeeded ✅" (or failed ❌)
```

---

### Phase 3: Rollback (If Something Goes Wrong)

```
Trigger: Verify step detects degradation OR GitOpsSync fails OR any step fails

failureStrategy: StageRollback triggers automatically:

├── Step R1: RevertPR
│   ├── Takes the commitId from Step 1 (UpdateReleaseRepo)
│   ├── Creates a NEW commit that undoes the image tag changes
│   ├── Opens a revert PR
│   └── values.yaml goes back to previous tags (old version)
│
├── Step R2: MergePR (revert)
│   ├── Merges the revert PR into main
│   ├── Now main has the OLD image tags again
│   └── Deletes source branch
│
└── Step R3: GitOpsSync (rollback)
    ├── Tells ArgoCD: "sync NOW"
    ├── ArgoCD reads main → sees old tags → deploys old version
    ├── Old pods come back → healthy
    └── Users never affected (< 5 min total)

Result: Old working version is back. Git history shows exactly what happened.
```

---

### Phase 4: Observability (Always Running)

```
ArgoCD auto-syncs these (self-healing):

Prometheus (kube-prometheus-stack Helm):
├── Scrapes all pods every 15s
├── Stores metrics for 15 days (50Gi PVC)
├── Feeds Grafana dashboards
└── Feeds Harness Verify step (Continuous Verification)

Grafana:
├── Pre-configured Prometheus datasource
├── Access: grafana.yourdomain.com (Ingress → shared ALB)
└── Login: admin / admin123

EFK (Elasticsearch + Fluentd + Kibana):
├── Fluentd DaemonSet → collects ALL container logs
├── Ships to Elasticsearch (X-Pack security, 10Gi PVC)
├── Kibana: kibana.yourdomain.com (Ingress → shared ALB)
└── Login: elastic / HarnessEFK@2026

Jaeger + OTel Collector:
├── Microservices send traces → otel-collector:4317 (gRPC)
├── OTel exports traces → Jaeger
├── OTel exports span metrics → Prometheus
├── Jaeger UI: jaeger.yourdomain.com (Ingress → shared ALB)
└── Shows: request flow across all 11 services

All via 1 ALB (cost optimized):
├── app.yourdomain.com → frontend
├── grafana.yourdomain.com → Grafana
├── kibana.yourdomain.com → Kibana
└── jaeger.yourdomain.com → Jaeger
```

---

### Phase 5: Secrets Flow

```
You add secrets in AWS Console (Secrets Manager):
└── online-boutique/app-secrets → { REDIS_ADDR: "redis-cart:6379", ... }

External Secrets Operator (runs in cluster):
├── Polls AWS SM every 1 hour
├── Reads online-boutique/app-secrets
├── Creates K8s Secret "app-secrets" in online-boutique namespace
└── Pods reference it: envFrom: secretRef: app-secrets

To update a secret:
1. Change value in AWS Console → Secrets Manager
2. Wait 1 hour (or delete ExternalSecret to force refresh)
3. Pods automatically get new value on next restart
```

---

### Phase 6: Governance (OPA — Automatic)

```
Every time pipeline runs → OPA evaluates:

Rule 1: Is it Friday? → BLOCK (no Friday deploys)
Rule 2: Does pipeline have HarnessApproval step? → required
Rule 3: Does pipeline have rollbackSteps? → required
Rule 4: Any hardcoded AWS account ID? → BLOCK
Rule 5: Using KubernetesDirect (not Harness Cloud)? → required

If ANY rule fails → Pipeline BLOCKED before execution starts.
No human intervention needed. Governance automated.
```

---

### Summary: What Happens When You Push Code

```
Developer pushes code to main
         ↓
Pipeline auto-runs (or manual trigger)
         ↓
OPA evaluates → PASS (not Friday, has approval, has rollback)
         ↓
Security scans (4 tools parallel) → results saved
         ↓
Build 11 Docker images (parallel) → push to ECR
         ↓
AI Security Agent → "2 MEDIUM vulns, 0 CRITICAL — safe"
         ↓
AI Risk Agent → "SAFE ✅ — proceed"
         ↓
GitOps: Update values.yaml (PR with new image tags)
         ↓
Human approves PR in Harness UI
         ↓
PR merged → ArgoCD syncs → new version deployed (rolling, zero downtime)
         ↓
Verify: Prometheus metrics compared → no degradation
         ↓
Slack: "Deploy #42 succeeded ✅"
         ↓
If ANYTHING fails → auto-rollback (revert PR → sync old version)
```

This is the MNC production pattern. One `terraform apply` sets it up, then every code push follows this flow automatically.
