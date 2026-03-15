
# CI/CD Pipeline RBAC Lab

# GitHub Actions → Kubernetes Deployment

## Lab Goal

We will create a **ServiceAccount for CI/CD**.

The pipeline will be able to:

✅ Deploy applications
✅ Update deployments
✅ View pods

But it will **NOT have full cluster admin access**.

This is exactly how **real companies secure Kubernetes pipelines**.

---

# Architecture of the Lab

Explain this diagram in your video:

```
GitHub Repository
       │
       │ GitHub Actions Pipeline
       ▼
ServiceAccount (cicd-deployer)
       │
RoleBinding
       │
Role
       │
Permissions
(create deployments, update deployments, get pods)
       ▼
Kubernetes Cluster
```

---

# Step 1 — Create Namespace for Applications

Create a namespace where the pipeline will deploy apps.

```bash
kubectl create namespace dev-app
```

Verify:

```bash
kubectl get ns
```

Expected output:

```
dev-app
```

---

# Step 2 — Create ServiceAccount for CI/CD

The pipeline will use this account.

```bash
kubectl create serviceaccount cicd-deployer -n dev-app
```

Verify:

```bash
kubectl get serviceaccounts -n dev-app
```

Output:

```
cicd-deployer
```

Explain in your video:

This ServiceAccount represents **our CI/CD pipeline identity**.

---

# Step 3 — Create RBAC Role for Deployment

We create a role that allows the pipeline to deploy applications.

Create file:

### `cicd-role.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-app
  name: cicd-deployer-role
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get","list","create","update","patch"]

- apiGroups: [""]
  resources: ["pods","services"]
  verbs: ["get","list","watch"]
```

---

### Explain permissions

Deployments permissions:

```
create
update
patch
```

Meaning pipeline can:

* deploy applications
* update deployments

Pods permissions:

```
get
list
watch
```

Meaning pipeline can monitor deployments.

---

# Step 4 — Apply Role

```bash
kubectl apply -f cicd-role.yaml
```

Verify:

```bash
kubectl get roles -n dev-app
```

Expected:

```
cicd-deployer-role
```

---

# Step 5 — Create RoleBinding

Now connect the ServiceAccount to the Role.

Create file:

### `cicd-rolebinding.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cicd-deployer-binding
  namespace: dev-app
subjects:
- kind: ServiceAccount
  name: cicd-deployer
  namespace: dev-app
roleRef:
  kind: Role
  name: cicd-deployer-role
  apiGroup: rbac.authorization.k8s.io
```

---

Apply:

```bash
kubectl apply -f cicd-rolebinding.yaml
```

Verify:

```bash
kubectl get rolebindings -n dev-app
```

Output:

```
cicd-deployer-binding
```

---

# Step 6 — Test RBAC Permissions

Test if the CI/CD user can deploy.

### Test create deployment

```bash
kubectl auth can-i create deployments \
--as system:serviceaccount:dev-app:cicd-deployer \
-n dev-app
```

Expected:

```
yes
```

---

### Test delete namespace (should fail)

```bash
kubectl auth can-i delete namespace \
--as system:serviceaccount:dev-app:cicd-deployer
```

Expected:

```
no
```

This proves RBAC security works.

---

# Step 7 — Create Application Manifest

Create a Kubernetes deployment file.

### `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
  namespace: dev-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
      - name: demo
        image: nginx
        ports:
        - containerPort: 80
```

---

# Step 8 — Deploy Application (Simulating Pipeline)

Run:

```bash
kubectl apply -f deployment.yaml
```

Check deployment:

```bash
kubectl get deployments -n dev-app
```

Check pods:

```bash
kubectl get pods -n dev-app
```

---



