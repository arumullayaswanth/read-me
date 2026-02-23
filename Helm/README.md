
# 🎬 VIDEO TITLE

**Helm Complete Masterclass | Beginner to Advanced + Real E-Commerce Deployment & Helm Repository**

---

# 🎤 FULL END-TO-END SCRIPT (READ DIRECTLY)

---

# 🟢 INTRO – SET THE STORY

Hello everyone 👋

Today is not just another Helm tutorial.

Today, I want to build something real with you.

Not just commands.
Not just theory.
But a complete E-Commerce application using Helm… step by step… from zero to advanced level.

Imagine this.

You joined a company as a DevOps engineer.
Developers come to you and say:

“We have two microservices ready — Cart Service and Checkout Service.
We want them deployed on Kubernetes.
We want version control.
We want easy upgrades.
We want rollbacks.
We want production-ready deployment.”

Now the question is…

Will you write 20 YAML files manually every time?

Or will you use Helm like a professional?

Today I will show you how professionals do it.

Let’s begin.

---

# 🟢 PART 1 – Problem with Raw YAML Files

Imagine this.

You deploy one microservice.

You write:

* deployment.yaml
* service.yaml
* configmap.yaml
* ingress.yaml
* secrets.yaml

Now imagine you have 5 microservices.

Now imagine you have dev, staging, and production.

Now you are managing 75+ YAML files.

Developer says:
“Change replica count to 3.”

You open files.
Edit them.
Hope nothing breaks.

Now developer says:
“Rollback to previous version.”

How?

This is the real pain.

Problems with raw YAML:

• Repetition
• Hardcoded values
• No structured release management
• No built-in rollback
• Environment management becomes messy

This is where Helm was introduced.

---

# 🟢 PART 2 – What is Helm?


Helm is a package manager for Kubernetes.

Just like:

* apt for Ubuntu
* npm for Node.js
* yum for RHEL

Helm allows us to package Kubernetes resources into something called a **Helm Chart**.

Instead of applying raw YAML,
we install a chart.

Helm gives:

• Reusability
• Versioning
• Rollback support
• Structured deployments

---

# 🟢 PART 3 – Helm vs kubectl

Let’s remove confusion early.

kubectl:

```bash
kubectl apply -f deployment.yaml
```

Helm:

```bash
helm install myapp .
```

kubectl:
• Applies raw YAML
• No release tracking
• No templating
• No rollback

Helm:
• Templating engine
• Release management
• Upgrade system
• Rollback capability
• Dependency management

kubectl is low-level.

Helm is lifecycle management.

Both are important. But Helm makes life easier.

---

# 🟢 PART 4 – Install Helm & Public Repo Demo

Install Helm:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Check:

```bash
helm version
```

Add public repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
```bash
What is Bitnami?

Bitnami is a company that provides pre-packaged, production-ready applications.

They create and maintain high-quality Helm charts for popular software like:

Nginx

MySQL

PostgreSQL

Redis

Kafka

WordPress

MongoDB

Instead of writing YAML for these from scratch,
we can install them directly using Bitnami charts.

🟢 Why Bitnami is Important

In real companies:

You don’t write your own Redis YAML.

You don’t manually configure PostgreSQL every time.

You use trusted, well-maintained Helm charts.

Bitnami charts are:

• Production-ready
• Security updated
• Well documented
• Actively maintained

That’s why they are widely used.
```


Search:

```bash
helm search repo nginx
```

Install nginx:

```bash
helm install mynginx bitnami/nginx
```
```bash
Helm:

Downloads chart from Bitnami repo

Reads default values.yaml

Renders templates

Sends to Kubernetes

Stores release

You didn’t write a single YAML file.

But you deployed a full application.

That is powerful.
```

Check:

```bash
helm list
kubectl get pods
```

Uninstall:

```bash
helm uninstall mynginx
```

Now students see Helm working.

---

# 🟢 PART 5 – Helm Architecture (Internal Working)

![Image](https://glasskube.dev/assets/images/helm-workflow-diagram-73ec11046f99e2e990ce3cabc5b6105c.png)

When you run:

```bash
helm install myapp .
```

Internally:

1. Helm reads Chart.yaml
2. Reads values.yaml
3. Processes templates
4. Generates final YAML
5. Sends to Kubernetes API
6. Stores release metadata as Secret

That’s why we can do:

```bash
helm history myapp
helm rollback myapp 1
```

Helm = Rendering Engine + Release Manager.

---

# 🟢 PART 6 – Create Your Own Helm Chart

```bash
helm create ecommerce
cd ecommerce
```

Now list structure.

You will see:

* Chart.yaml
* values.yaml
* templates/
* charts/
* .helmignore

Let’s understand each one.

---

# 🟢 PART 7 – Helm Chart Structure Deep Dive

## Chart.yaml

Identity of chart.
Contains name, version, appVersion.

## values.yaml

Configuration file.
Controls replicas, image, service type.

## templates/

Kubernetes YAML templates with variables.

## _helpers.tpl

Reusable template logic.

## charts/

Dependencies or subcharts.

## .helmignore

Files to ignore when packaging.

Now structure is clear.

---

### 🟢 1️⃣ Chart.yaml – Identity of the Chart

Think of Chart.yaml like an Aadhaar card for your Helm chart.

It defines:

* Name
* Version
* Description
* App Version

Open Chart.yaml.

You’ll see something like:

```yaml
apiVersion: v2
name: mychart
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.16.0"
```

Let me explain clearly:

* apiVersion → Helm chart format version
* name → Chart name
* version → Chart version (not app version)
* appVersion → Application version

Very important concept:

If developer updates Docker image version → change appVersion.
If you change chart structure → change version.

This is how professionals manage versions.

---

### 🟢 2️⃣ values.yaml – The Brain of the Chart

This is the most important file.

values.yaml stores all configurable values.

Open values.yaml.

You will see:

* replicaCount
* image repository
* service type
* resources

Now understand this carefully:

Instead of hardcoding values in deployment.yaml,
we use:

```yaml
{{ .Values.replicaCount }}
```

Helm replaces values dynamically.

Think of values.yaml like settings panel.

If you want to:

* Change replicas
* Change image version
* Change service type

You don’t edit templates.

You only change values.yaml.

That is clean design.

---

### 🟢 3️⃣ templates/ – Where Kubernetes YAML Lives

![Image](https://razorops.com/images/blog/helm-3-tree.png)
![Image](https://devopscube.com/content/images/2025/03/helm-template-1.png)


Now go inside templates folder.

```bash
cd templates
ls
```

Here you will see:

* deployment.yaml
* service.yaml
* ingress.yaml
* _helpers.tpl
* tests/

This folder contains Kubernetes manifests.

But they are not normal YAML files.

They are templates.

Inside deployment.yaml, you will see:

```yaml
replicas: {{ .Values.replicaCount }}
```

This is Go templating.

When we run:

```bash
helm install myrelease .
```

Helm:

1. Reads values.yaml
2. Replaces template variables
3. Generates final YAML
4. Sends it to Kubernetes

That is the Helm engine.

---

### 🟢 4️⃣ _helpers.tpl – Reusable Template Logic

Now this is advanced but very powerful.

_helpers.tpl is used for reusable template code.

Example:

```yaml
{{- define "mychart.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end }}
```

Then inside deployment.yaml you may see:

```yaml
name: {{ include "mychart.fullname" . }}
```

What is happening?

Instead of repeating naming logic everywhere,
we define it once in helpers file.

This is clean architecture.

Think of it like creating a function in programming.

Reusable. Maintainable. Professional.

---

### 🟢 5️⃣ charts/ – Subcharts & Dependencies

Now look at charts/ folder.

This folder is used when:

Your application depends on another chart.

Example:

Your app needs:

* Redis
* MySQL
* PostgreSQL

Instead of writing those manually,
you can include them as subcharts.

charts/ folder stores dependency charts.

In production, microservices often use this.

This is how complex systems are built cleanly.

---

### 🟢 6️⃣ .helmignore – Ignore Unwanted Files

This works like .gitignore.

If you don’t want certain files included when packaging:

You add them in .helmignore.

Example:

```bash
.git/
README.md
```

When you run:

```bash
helm package .
```

Ignored files won’t be included.

Small file, but very important in production.

---

### 🟢 HOW EVERYTHING CONNECTS

Let’s summarize like professionals.

When you run:

```bash
helm install myrelease .
```

Here is what happens internally:

1. Helm reads Chart.yaml
2. Reads values.yaml
3. Reads templates/
4. Processes helpers
5. Generates final YAML
6. Sends to Kubernetes API

Helm is basically a rendering engine + release manager.

Now you understand what happens behind the scenes.


---

# 🟢 PART 8 – values.yaml Deep Dive


---

# 🟢 PART 9 – Helm Templating Basics


---

# 🟢 PART 10 – Helm Lifecycle

Install:

```bash
helm install ecommerce-release .
```

Upgrade:

```bash
helm upgrade ecommerce-release .
```

History:

```bash
helm history ecommerce-release
```

Rollback:

```bash
helm rollback ecommerce-release 1
```

Uninstall:

```bash
helm uninstall ecommerce-release
```

This is production power.

---

# 🟢 PART 11 – REAL PROJECT (E-Commerce Microservices)


# 🟢 PART 12 – Package the Chart


# 🟢 PART 13 – Create Your Own Helm Repository

# 🟢 PART 14 – Advanced Overview



---

# 🎬 FINAL CLOSE

Today we did not just learn commands.

We learned:

• Why Helm exists
• How Helm works internally
• How to create structured charts
• How to manage releases
• How to deploy microservices
• How to package and host charts

If you understood everything here…

You are ready for production-level Helm.

Keep building.
Keep improving.
See you in the next masterclass 🚀


