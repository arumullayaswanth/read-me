

# Kubernetes RBAC Hands-On Lab (Step-by-Step)

## Lab Scenario

Imagine this situation:

You have a **developer team**.

Developers should only be able to:

✅ View pods
✅ List pods

But they should **NOT be able to delete pods**.

We will implement this using **RBAC**.

---
# Step 8 — Visualize the Flow

What we created:

```
ServiceAccount
      │
      │
RoleBinding
      │
      │
Role
      │
Permissions
(get pods, list pods)
```

RBAC allows only **specific actions**.

---

# Step 1 — Start Your Kubernetes Cluster

Make sure your cluster is running.

You can use:

* Minikube
* Kind
* EKS
* AKS
* GKE

Check cluster:

```bash
kubectl cluster-info
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```
NAME           STATUS   ROLES           AGE
minikube       Ready    control-plane   2d
```

---

# Step 2 — Create a Namespace

We will create a namespace called **dev-team**.

```bash
kubectl create namespace dev-team
```

Verify:

```bash
kubectl get namespaces
```

Expected output:

```
dev-team
default
kube-system
```

Now everything in this lab will happen **inside dev-team namespace**.

---

# Step 3 — Create a Sample Deployment

We need some pods to test RBAC.

Run:

```bash
kubectl create deployment nginx \
--image=nginx \
--namespace=dev-team
```

Check pods:

```bash
kubectl get pods -n dev-team
```

Expected output:

```
nginx-xxxxx
```

Good — now we have a resource to test.

---

# Step 4 — Create a Role

Now we will create a **Role**.

This role will allow:

* get pods
* list pods
* watch pods

Create a file.

### File name

```
role-pod-reader.yaml
```

Content:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-team
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

Explanation (important for YouTube):

```
apiGroups: ""
means core API group

resources:
pods

verbs:
get → read one pod
list → see all pods
watch → monitor pods
```

---

Apply it:

```bash
kubectl apply -f role-pod-reader.yaml
```

Verify:

```bash
kubectl get roles -n dev-team
```

Expected:

```
pod-reader
```

---

# Step 5 — Create a ServiceAccount

Instead of a normal user, we will create a **service account**.

This represents a **developer application**.

Command:

```bash
kubectl create serviceaccount dev-user -n dev-team
```

Verify:

```bash
kubectl get serviceaccounts -n dev-team
```

Expected:

```
dev-user
```

---

# Step 6 — Create RoleBinding

Now we connect:

ServiceAccount → Role

Create a file.

### File name

```
rolebinding-dev.yaml
```

Content:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
  namespace: dev-team
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: dev-team
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Explanation:

subjects → who gets permission
roleRef → which role they get

---

Apply:

```bash
kubectl apply -f rolebinding-dev.yaml
```

Verify:

```bash
kubectl get rolebindings -n dev-team
```

Expected:

```
dev-user-binding
```

---

# Step 7 — Test RBAC Permissions

Now we will test if permissions work.

Use:

```bash
kubectl auth can-i
```

---

### Test 1 — Can read pods

```bash
kubectl auth can-i get pods \
--as system:serviceaccount:dev-team:dev-user \
-n dev-team
```

Expected output:

```
yes
```

---

### Test 2 — Can list pods

```bash
kubectl auth can-i list pods \
--as system:serviceaccount:dev-team:dev-user \
-n dev-team
```

Expected:

```
yes
```

---

### Test 3 — Can delete pods

```bash
kubectl auth can-i delete pods \
--as system:serviceaccount:dev-team:dev-user \
-n dev-team
```

Expected:

```
no
```

This means RBAC is working correctly.

---


---

# Step 9 — Check Everything

Run these commands to show your audience:

```bash
kubectl get roles -n dev-team
```

```
pod-reader
```

---

```bash
kubectl get rolebindings -n dev-team
```

```
dev-user-binding
```

---

```bash
kubectl get serviceaccounts -n dev-team
```

```
dev-user
```

---



---

# Step 11 — Cleanup (Optional)

Delete everything:

```bash
kubectl delete namespace dev-team
```

---

# What You Achieved

You successfully implemented:

✅ Namespace
✅ Deployment
✅ Role
✅ ServiceAccount
✅ RoleBinding
✅ Permission testing

This is the **foundation of Kubernetes RBAC**.

---
.
