
## Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│  TERRAFORM (One Apply = Entire Platform)                              │
│                                                                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────────┐  ┌────────┐  │
│  │  VPC    │→ │  EKS    │→ │Bastion  │→ │ Delegate  │→ │GitOps  │  │
│  │ Subnets │  │AutoMode │  │  SSM    │  │  HA (2)   │  │Agent   │  │
│  │ NAT, RT │  │ KMS,IAM │  │SonarQube│  │Autoscale  │  │HA (2)  │  │
│  └─────────┘  └─────────┘  └─────────┘  └───────────┘  └────────┘  │
│                                                                       │
│  ┌─────────┐  ┌─────────────┐  ┌────────────┐  ┌─────────────────┐  │
│  │  ECR    │  │ Harness     │  │ Harness    │  │  External       │  │
│  │11 repos │  │ Service     │  │ Environment│  │  Secrets        │  │
│  │lifecycle│  │(ReleaseRepo)│  │ (dev+prod) │  │  Operator       │  │
│  └─────────┘  └─────────────┘  └────────────┘  └─────────────────┘  │
│                                                                       │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐  ┌────────────┐  │
│  │ Connectors  │  │ OPA Policy   │  │ Monitored  │  │Kong Gateway│  │
│  │Prometheus   │  │ + PolicySet  │  │ Service    │  │ 3.9 OSS    │  │
│  │K8s, ELK     │  │ (On Run)     │  │(CV — 3 HS) │  │NLB + ACM   │  │
│  └─────────────┘  └──────────────┘  └────────────┘  └────────────┘  │
│                                                                       │
├──────────────────────────────────────────────────────────────────────┤
│  KONG GATEWAY 3.9 (API Gateway + Ingress Controller)                 │
│                                                                       │
│  NLB (internet-facing, ACM TLS termination on 443)                   │
│  ├── kong.domain    → Kong Manager OSS UI (basic-auth protected)     │
│  ├── grafana.domain → Prometheus + Grafana (login: admin/password)   │
│  ├── kibana.domain  → Kibana (login: elastic/password via xpack)     │
│  ├── jaeger.domain  → Jaeger Query UI (open access)                  │
│  └── app.domain     → Online Boutique Frontend                       │
│  20 Global Plugins: rate-limit, CORS, Zipkin, bot-detect, JWT, etc.  │
│                                                                       │
├──────────────────────────────────────────────────────────────────────┤
│  HARNESS ENTERPRISE PIPELINE (Per Code Push)                          │
│                                                                       │
│  Stage 1: CI (KubernetesDirect — code stays in VPC)                  │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │ [Parallel] Gitleaks │ Trivy │ OWASP │ SonarQube             │    │
│  │            ↓ scan results saved as JSON                       │    │
│  │ [Parallel] BuildAndPushECR × 11 services (OIDC, tag: v#)    │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                       │
│  Stage 2: AI Security Agent                                          │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │ python security_agent.py → reads scan JSONs → AI report      │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                       │
│  Stage 3: AI Deployment Risk Agent                                   │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │ python deployment_risk_agent.py → SAFE/RISKY/BLOCK           │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                       │
│  Stage 4: GitOps Deploy (gitOpsEnabled: true)                        │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │ UpdateReleaseRepo → Approval → MergePR → GitOpsSync → Verify│    │
│  │      (updates 11        (human       (merge to   (ArgoCD     │    │
│  │       image tags)        review)      main)      syncs)      │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                       │
│  Rollback (auto on failure):                                         │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │ RevertPR → MergePR (revert) → GitOpsSync (old version)      │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                       │
├──────────────────────────────────────────────────────────────────────┤
│  OBSERVABILITY (ArgoCD Helm Apps — auto-sync + self-heal)            │
│                                                                       │
│  Prometheus + Grafana → kube-prometheus-stack 62.3.0 (auto-sync)     │
│  Elasticsearch 7.17   → elastic Helm chart (xpack trial, auth)       │
│  Kibana 7.17          → elastic Helm chart (native login screen)     │
│  Fluentd              → fluent Helm chart (DaemonSet, auto-sync)     │
│  Jaeger 3.1.1         → jaeger Helm chart (badger storage)           │
│  OTel Collector 0.97  → opentelemetry Helm chart (deployment mode)   │
│  All via Kong Ingress → 1 NLB → subdomains (grafana., kibana., etc.) │
│                                                                       │
├──────────────────────────────────────────────────────────────────────┤
│  SECRETS (External Secrets Operator + Pod Identity)                   │
│                                                                       │
│  AWS Secrets Manager → ESO → K8s Secret → Pods (auto-refresh 1h)   │
│  Passwords auto-generated: Grafana, EFK, Kong Admin, JWT, RDS        │
│  No secrets in Git. No Harness runtime resolution needed.            │
│                                                                       │
├──────────────────────────────────────────────────────────────────────┤
│  GOVERNANCE (OPA Policy — evaluated On Run)                          │
│                                                                       │
│  Rules: No Friday deploys │ Require approval │ Require rollback     │
│         No hardcoded IDs  │ Must use KubernetesDirect               │
│                                                                       │
├──────────────────────────────────────────────────────────────────────┤
│  CONTINUOUS VERIFICATION (3 Health Sources — auto-rollback)           │
│                                                                       │
│  Monitored Service: online-boutique-production                       │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │ Health Source 1: Prometheus (Metrics — infrastructure)           │ │
│  │   Query: kube_pod_container_status_restarts_total                │ │
│  │   Detects: Pod crash-loops, restarts after deploy                │ │
│  ├─────────────────────────────────────────────────────────────────┤ │
│  │ Health Source 2: Prometheus (Metrics — HTTP 5xx via Kong)        │ │
│  │   Query: kong_http_requests_total{code=~"5.."}                   │ │
│  │   Detects: App returning 502/503/500 to real users               │ │
│  ├─────────────────────────────────────────────────────────────────┤ │
│  │ Health Source 3: Elasticsearch (Logs — application errors)       │ │
│  │   Index: fluentd-*  Query: namespace:online-boutique AND error   │ │
│  │   Detects: Exceptions, stack traces, error log spikes            │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                       │
│  Verify Step (5 min, MEDIUM sensitivity):                            │
│    Compares pre-deploy vs post-deploy for ALL 3 health sources       │
│    ANY anomaly detected → automatic rollback (RevertPR → Sync)       │
│                                                                       │
├──────────────────────────────────────────────────────────────────────┤
│  NOTIFICATIONS                                                        │
│                                                                       │
│  Slack: Pipeline success/failure                                     │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Pipeline Steps Explained

| # | Step | Type | What It Does |
|---|------|------|-------------|
| **Stage 1: security-scans** | | **CI (KubernetesDirect)** | **DevSecOps — 5 parallel security scans** |
| 1.1 | Gitleaks Scan | Run (parallel) | Detects hardcoded secrets/passwords in source code. Image: `zricethezav/gitleaks:latest`. Outputs JSON report. Fails if leaks found. |
| 1.2 | Trivy Scan | Run (parallel) | Filesystem vulnerability scan (HIGH+CRITICAL). Image: `aquasec/trivy:latest`. Scans `Episode-10/src/` directory. |
| 1.3 | OSV Dependency Scan | Run (parallel) | Google OSV scanner — checks all dependencies for known CVEs. Recursive scan of all package manifests. |
| 1.4 | SonarQube Analysis | Run (parallel) | Code quality + security analysis. Connects to SonarQube on Bastion (`<+variable.sonar_host_url>`). 3Gi memory. |
| 1.5 | Checkov IaC Scan | Run (parallel) | Infrastructure-as-Code security. Scans Terraform + K8s manifests for misconfigurations (uses `.checkov.yaml` for suppressions). |
| **Stage 2: build-and-push** | | **CI (KubernetesDirect)** | **Build 11 microservice Docker images → Push to ECR** |
| 2.1 | Restore Cache | RestoreCacheS3 | Restores Go modules, npm packages, Gradle deps from S3 (speeds up builds). |
| 2.2 | Push Frontend | BuildAndPushECR (batch 1) | Go multi-stage build. Tag: `v<sequenceId>`. Remote cache in ECR `cache` repo. 4Gi RAM. |
| 2.3 | Push CartService | BuildAndPushECR (batch 1) | .NET build. 4Gi RAM. |
| 2.4 | Push CheckoutService | BuildAndPushECR (batch 1) | Go build. 3Gi RAM. |
| 2.5 | Push ProductCatalog | BuildAndPushECR (batch 2) | Go build. 3Gi RAM. |
| 2.6 | Push CurrencyService | BuildAndPushECR (batch 2) | Node.js build. 2Gi RAM. |
| 2.7 | Push EmailService | BuildAndPushECR (batch 2) | Python build. 2Gi RAM. |
| 2.8 | Push PaymentService | BuildAndPushECR (batch 3) | Node.js build. 2Gi RAM. |
| 2.9 | Push RecommendationService | BuildAndPushECR (batch 3) | Python build. 2Gi RAM. |
| 2.10 | Push ShippingService | BuildAndPushECR (batch 3) | Go build. 3Gi RAM. |
| 2.11 | Push AdService | BuildAndPushECR (batch 4) | Java/Gradle build. 4Gi RAM (heaviest). |
| 2.12 | Push LoadGenerator | BuildAndPushECR (batch 4) | Python build. 2Gi RAM. |
| 2.13 | Prepare Cache Dirs | Run | Creates cache directories for S3 save step. |
| 2.14 | Save Cache | SaveCacheS3 | Saves Go/npm/Gradle caches to S3 for next run. |
| **Stage 3: ai-agents** | | **CI (KubernetesDirect)** | **5 AI Agents (Google Gemini) analyze the deployment** |
| 3.1 | AI Security Agent | Run (parallel) | Analyzes pipeline security scan context. Generates AI-powered security assessment report. |
| 3.2 | AI Code Review Agent | Run (parallel) | Reviews recently changed files (last 5 commits). Finds bugs, security issues, performance problems. Gives score 1-10. |
| 3.3 | AI Release Notes Agent | Run (parallel) | Analyzes last 20 git commits. Generates release notes with contributors, commit table, categorized changes. |
| 3.4 | AI Deployment Risk Agent | Run (parallel) | Evaluates risk: environment, test status, file count, time of day, vulnerabilities. Returns SAFE/RISKY/BLOCK. |
| 3.5 | AI Log Analysis Agent | Run (parallel) | Analyzes recent git log for patterns. Identifies risky changes and deployment concerns. |
| **Stage 4: gitops-deploy** | | **Deployment (GitOps)** | **Production deploy via ArgoCD (PR → Approve → Merge → Sync → Verify)** |
| 4.1 | Update Release Repo | GitOpsUpdateReleaseRepo | Creates PR updating all 11 image tags in `values.yaml`. PR title: "Deploy Online Boutique build #N". |
| 4.2 | Approve Deployment | HarnessApproval | Human approval gate. Any project user can approve. Timeout: 1 day. |
| 4.3 | Merge PR | MergePR | Merges the PR into main branch. Deletes source branch. ArgoCD watches main. |
| 4.4 | Sync Application | GitOpsSync | Triggers immediate ArgoCD sync (prune enabled). Doesn't wait for 3-min poll interval. |
| 4.5 | Get App Status | GitOpsGetAppDetails | Checks ArgoCD app health: Synced/Healthy/Degraded/Progressing. |
| 4.6 | Verify Deployment | Verify (CV) | Continuous Verification — 5 min, LOW sensitivity. Compares pre vs post deploy metrics (Prometheus: pod restarts, CPU, memory) + logs (Elasticsearch: error patterns). ANY anomaly → triggers rollback. |
| **Rollback (auto on Stage 4 failure)** | | **Auto** | **Reverts Git commit → ArgoCD syncs old version** |
| R1 | Revert PR | RevertPR | Creates revert of the deploy commit from step 4.1. |
| R2 | Merge Revert PR | MergePR | Merges revert into main. Old image tags restored in values.yaml. |
| R3 | Rollback Sync | GitOpsSync | ArgoCD syncs reverted state → previous working version deployed. |
| **Stage 5: dast-owasp-zap** | | **CI (KubernetesDirect)** | **DAST — Scans LIVE deployed app for web vulnerabilities** |
| 5.1 | OWASP ZAP Baseline Scan | Run | Spiders `https://app.<domain>` for 1 min. Passive scans all responses. Outputs HTML+JSON reports. Uploads HTML to S3 with presigned URL (7-day access). 4Gi RAM. |
| **Notifications** | | **Slack** | **Pipeline success/failure alerts to Slack channel** |

---
