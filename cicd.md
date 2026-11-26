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


## **Pipeline 4: Functional Testing Pipeline**

| Stage / Tool           | Job                     | Easy meaning                                                       |
| ---------------------- | ----------------------- | ------------------------------------------------------------------ |
| Checkout               | Source code             | Downloads application code from GitHub                             |
| Install Dependencies   | Dependency installation | Installs required packages for testing                             |
| Start Application      | Application startup     | Starts the application in the test environment                     |
| Test Environment Setup | Environment setup       | Prepares the required test environment and configurations          |
| API Testing            | API validation          | Checks whether APIs work as expected                               |
| UI Testing             | Frontend testing        | Checks whether buttons, forms, pages and user interactions work    |
| Functional Tests       | Functional validation   | Checks whether application features work according to requirements |
| Integration Tests      | Integration testing     | Checks whether different application components work together      |
| Database Validation    | Database testing        | Checks whether data is stored and retrieved correctly              |
| Test Reports           | Reporting               | Generates test execution reports                                   |
| Publish Test Results   | Test results            | Uploads and displays test reports                                  |
| Cleanup                | Resource cleanup        | Stops the application and removes temporary test resources         |

### **Common Functional Testing Tools**

| Tool            | Job                       | Easy meaning                                 |
| --------------- | ------------------------- | -------------------------------------------- |
| Selenium        | UI automation             | Automates browser testing                    |
| Playwright      | UI / E2E testing          | Tests complete user workflows in browsers    |
| Cypress         | Frontend testing          | Tests frontend application functionality     |
| Postman         | API testing               | Tests API requests and responses             |
| REST Assured    | API automation            | Automates REST API testing                   |
| Pytest          | Test automation           | Runs Python functional and integration tests |
| JUnit           | Unit / functional testing | Runs Java application tests                  |
| Robot Framework | Acceptance testing        | Automates business-level test cases          |

**Example workflow:**

`Checkout → Install Dependencies → Start Application → API Tests → UI Tests → Functional Tests → Generate Reports → Cleanup`

**Real-time example:** When a user registers on your application, functional tests verify that registration works, the user details are saved in PostgreSQL, and the user can log in successfully.


## **Pipeline 5: Performance Testing Pipeline**

| Stage / Tool                 | Job                       | Easy meaning                                                       |
| ---------------------------- | ------------------------- | ------------------------------------------------------------------ |
| Checkout                     | Source code               | Downloads application code from GitHub                             |
| Install Dependencies         | Dependency installation   | Installs required packages for performance testing                 |
| Test Environment Setup       | Environment setup         | Prepares the test environment                                      |
| Start Application            | Application startup       | Starts the application before testing                              |
| Load Testing                 | Load testing              | Checks application performance under expected user traffic         |
| Stress Testing               | Stress testing            | Checks how the application behaves under extreme traffic           |
| Spike Testing                | Spike testing             | Checks how the application handles sudden increases in traffic     |
| Endurance Testing            | Soak testing              | Checks application stability under continuous load for a long time |
| API Performance Testing      | API benchmarking          | Measures API response time and throughput                          |
| Database Performance Testing | DB benchmarking           | Checks database query performance under load                       |
| Resource Monitoring          | Infrastructure monitoring | Monitors CPU, memory, disk and network usage                       |
| Response Time Analysis       | Performance analysis      | Measures average, minimum, maximum and percentile response times   |
| Generate Reports             | Test reporting            | Generates performance test reports                                 |
| Threshold Validation         | Performance validation    | Checks whether response time and error rate meet defined limits    |
| Publish Results              | Reporting                 | Publishes test results for the team                                |
| Cleanup                      | Resource cleanup          | Stops testing and removes temporary resources                      |

### **Common Performance Testing Tools**

| Tool              | Job                 | Easy meaning                                       |
| ----------------- | ------------------- | -------------------------------------------------- |
| JMeter            | Load testing        | Simulates multiple users accessing the application |
| k6                | Performance testing | Tests API and application performance              |
| Locust            | Load testing        | Simulates user traffic using Python                |
| Gatling           | Load testing        | Measures application performance under load        |
| Apache Bench (ab) | HTTP benchmarking   | Measures web server performance                    |
| wrk               | HTTP benchmarking   | Generates HTTP traffic to test server performance  |
| Prometheus        | Metrics monitoring  | Collects CPU, memory and application metrics       |
| Grafana           | Visualization       | Displays performance metrics in dashboards         |
| Node Exporter     | System monitoring   | Collects Linux server resource metrics             |

### **Performance Testing Flow**

`Checkout → Install Dependencies → Setup Environment → Start Application → Load Test → Stress Test → Monitor Resources → Analyze Results → Generate Report → Cleanup`

### **Real-time Example**

Suppose your application normally handles 1,000 users.

- Load testing: Check how it performs with 1,000 users.
- Stress testing: Increase traffic to 5,000 users and observe failures.
- Spike testing: Suddenly increase traffic from 100 to 5,000 users.
- Soak testing: Run 1,000 users continuously for 5–6 hours.

**Goal:** Make sure the application is fast, stable and scalable under different traffic conditions.



## **Pipeline 6: Synthetic Monitoring – Website Availability & Performance**

| Stage / Tool           | Job                     | Easy meaning                                                      |
| ---------------------- | ----------------------- | ----------------------------------------------------------------- |
| Synthetic Check        | Availability monitoring | Checks whether the application or website is accessible           |
| HTTP Check             | Endpoint monitoring     | Sends HTTP requests to check API or website availability          |
| DNS Check              | DNS monitoring          | Checks whether the domain name resolves correctly                 |
| SSL Certificate Check  | Certificate monitoring  | Checks SSL certificate validity and expiry                        |
| Response Time Check    | Performance monitoring  | Measures how much time the application takes to respond           |
| Status Code Validation | Health validation       | Checks whether the endpoint returns the expected HTTP status code |
| Login Flow Check       | User journey monitoring | Simulates a user login to verify the login flow                   |
| Alerting               | Incident notification   | Sends alerts when a synthetic check fails                         |

**Easy flow:**

`Synthetic Check → Website/API Request → Validate Response → Measure Response Time → Pass/Fail → Alert`

**Example:** Every 5 minutes, a synthetic monitoring tool checks your website. If the website returns HTTP 500 or takes too long to respond, it sends an alert to the DevOps team.

Common tools:

| Tool                         | Job                  | Easy meaning                              |
| ---------------------------- | -------------------- | ----------------------------------------- |
| Grafana Synthetic Monitoring | Synthetic checks     | Monitors website and API availability     |
| Pingdom                      | Uptime monitoring    | Checks whether a website is up or down    |
| Prometheus Blackbox Exporter | Endpoint monitoring  | Checks HTTP, HTTPS, DNS and TCP endpoints |


