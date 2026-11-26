## **Pipeline 1: CI/CD Pipeline**

| Stage / Tool          | Job               | Easy meaning                                                        |
| --------------------- | ----------------- | ------------------------------------------------------------------- |
| Checkout              | Source code       | Downloads code from GitHub                                          |
| Gitleaks              | Secrets scan      | Checks if secrets are written in code (API keys, passwords, tokens) |
| Lint                  | Code linting      | Checks code formatting and style                                    |
| Unit Tests            | Unit testing      | Tests individual parts of the application                           |
| SonarQube             | SAST              | Checks code quality and risky coding patterns                       |
| Trivy                 | Container scan    | Checks Docker images for vulnerabilities                            |
| Snyk                  | Dependency scan   | Checks `package.json` for old or vulnerable libraries               |
| Checkov               | IaC scan          | Checks Kubernetes YAML files for weak deployment configurations     |
| S3 Cache Restore      | Cache restore     | Restores previously saved cache                                     |
| Build Images          | Docker build      | Builds application Docker images                                    |
| S3 Cache Save         | Cache save        | Saves cache for future pipeline runs                                |
| Push to ECR           | Image registry    | Pushes Docker images to Amazon ECR                                  |
| OPA Policy Evaluation | Policy evaluation | Checks whether configurations follow defined policies               |
| Policy as Code        | Governance rules  | Writes governance rules using Rego language                         |
| Push to Git           | Git update        | Pushes updated files to Git                                         |

### **GitOps**

| Stage / Tool | Job                | Easy meaning                                       |
| ------------ | ------------------ | -------------------------------------------------- |
| GitOpsSync   | Synchronization    | Syncs the application state from Git to Kubernetes |
| GetAppStatus | Application status | Checks the current application status              |
| Verify (CV)  | Verification       | Verifies the deployment                            |

### **Rollback**

| Step | Action     | Easy meaning                                                 |
| ---- | ---------- | ------------------------------------------------------------ |
| 1    | RevertPR   | Creates a PR to revert the changes                           |
| 2    | MergePR    | Merges the revert PR                                         |
| 3    | GitOpsSync | Automatically syncs the reverted configuration to Kubernetes |

**Rollback flow:** `RevertPR → MergePR → GitOpsSync`

## **Pipeline 2: GitHub Actions – Ephemeral PostgreSQL Databases for CI**

| Stage | Job                                      | Easy meaning                                                    |
| ----- | ---------------------------------------- | --------------------------------------------------------------- |
| 1     | Check out code                           | Downloads project code from GitHub                              |
| 2     | Configure AWS credentials via OIDC       | Authenticates GitHub Actions with AWS using OIDC                |
| 3     | Load DB credentials from Secrets Manager | Loads database credentials from AWS Secrets Manager             |
| 4     | Install dependencies                     | Installs required project packages                              |
| 5     | Create ephemeral database                | Creates a temporary PostgreSQL database                         |
| 6     | Run migrations                           | Applies database schema changes                                 |
| 7     | Inspect database                         | Checks the database after migrations                            |
| 8     | Diff ephemeral DB vs base PostgreSQL DB  | Compares the temporary database with the base database          |
| 9     | Run integration tests                    | Tests application components working together with the database |
| 10    | Drop ephemeral database                  | Deletes the temporary database after testing                    |

## **Pipeline 3: Cron Job – Post-Deployment Security Monitoring**

| Tool      | Job              | Easy meaning                                                     |
| --------- | ---------------- | ---------------------------------------------------------------- |
| OWASP ZAP | DAST             | Checks the live website for common web security issues           |
| Falco     | Runtime security | Monitors the application and Kubernetes cluster after deployment |

