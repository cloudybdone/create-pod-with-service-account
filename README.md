# Kubernetes Pod with a Specific ServiceAccount

This lab demonstrates how to run a Kubernetes Pod using a **specific ServiceAccount** instead of relying on the namespace's default ServiceAccount.

The lab uses:

* **Namespace:** `dev1`
* **ServiceAccount:** `demo-sa`
* **Pod:** `demo`
* **Image:** `nginx`

---

## 🎯 Lab Objective

The objective of this lab is to understand how a Kubernetes Pod can be associated with a specific ServiceAccount.

The workflow is:

```text
Create Namespace
      ↓
Create ServiceAccount
      ↓
Create Pod
      ↓
Assign ServiceAccount
      ↓
Verify Pod Identity
      ↓
Cleanup
```

---
![Pod ServiceAccount](https://github.com/cloudybdone/create-pod-with-service-account/blob/main/serviceAccount.png)
# 🔐 What is a ServiceAccount?

A **ServiceAccount** provides an identity for processes running inside Kubernetes Pods.

It can be used when a workload needs to communicate with the Kubernetes API or access Kubernetes resources according to the permissions assigned to that identity.

In simple terms:

```text
Pod
 │
 │ runs as
 ▼
ServiceAccount
 │
 │ permissions controlled by
 ▼
RBAC
 │
 ▼
Kubernetes API / Resources
```

Kubernetes automatically creates a `default` ServiceAccount for each namespace. If a Pod is created without explicitly specifying a ServiceAccount, Kubernetes assigns the namespace's `default` ServiceAccount to that Pod.

This is important from a security perspective because the identity associated with a workload determines which permissions can potentially be granted to it.

---

# 1. Create the Namespace

First, create the `dev1` namespace:

```bash
kubectl create ns dev1
```

Verify that the namespace exists:

```bash
kubectl get namespace
```
![get-namespaces](https://github.com/cloudybdone/create-pod-with-service-account/blob/main/ss01.png)
The lab uses a dedicated namespace so that the ServiceAccount and Pod remain isolated from resources in other namespaces.

---

# 2. Create the ServiceAccount

Create a ServiceAccount named `demo-sa` inside the `dev1` namespace:

```bash
kubectl create serviceaccount demo-sa -n dev1
```

Verify the ServiceAccount:

```bash
kubectl get serviceaccounts -n dev1
```
![get-serviceAccount](https://github.com/cloudybdone/create-pod-with-service-account/blob/main/ss02.png)
Expected resource:

```text
demo-sa
```

At this stage, we have created the identity that will later be assigned to the Pod.

---

# 3. Create the Pod Using the Specific ServiceAccount

Now create the `demo` Pod using the `nginx` image and explicitly assign `demo-sa`.

```bash
kubectl run demo \
  --image=nginx \
  -n dev1 \
  --dry-run=client \
  -o json \
  --overrides='{"spec": {"serviceAccountName": "demo-sa"}}' \
  | kubectl apply -f -
```

### What is happening here?

The command first generates the Pod definition as JSON:

```bash
--dry-run=client -o json
```

Then this part:

```bash
--overrides='{"spec": {"serviceAccountName": "demo-sa"}}'
```

overrides the generated Pod specification and sets:

```yaml
spec:
  serviceAccountName: demo-sa
```

The resulting configuration is then passed to:

```bash
kubectl apply -f -
```

and applied to the cluster.

The important configuration is therefore:

```yaml
spec:
  serviceAccountName: demo-sa
```

This explicitly tells Kubernetes which ServiceAccount should be associated with the Pod.

---

# 4. Verify the Pod

Check whether the Pod is running:

```bash
kubectl get pods -n dev1
```

Expected:

```text
NAME    READY   STATUS    RESTARTS   AGE
demo    1/1     Running   0          ...
```

---

# 5. Verify the ServiceAccount

Now verify which ServiceAccount is associated with the Pod:

```bash
kubectl get pod demo -n dev1 -o yaml | grep serviceAccount
```
![get-podDemo](https://github.com/cloudybdone/create-pod-with-service-account/blob/main/ss03.png)
The output should contain:

```text
serviceAccount: demo-sa
```

This confirms that the Pod is using the explicitly specified `demo-sa` ServiceAccount rather than relying on the namespace's default ServiceAccount.

---

# 🔍 Understanding the Configuration

The important relationship in this lab is:

```text
Namespace: dev1
        │
        ├── ServiceAccount: demo-sa
        │
        └── Pod: demo
                 │
                 └── serviceAccountName: demo-sa
```

The Pod therefore runs with the identity:

```text
demo-sa
```

If we later assign RBAC permissions to `demo-sa`, those permissions can be associated with workloads using this ServiceAccount.

---

# 🧠 Key Takeaways

The main concept demonstrated by this lab is **workload identity**.

A Pod does not necessarily have to use the namespace's default ServiceAccount. We can explicitly assign a dedicated ServiceAccount:

```yaml
serviceAccountName: demo-sa
```

This becomes particularly important when designing Kubernetes security around:

* ServiceAccount
* RBAC
* Role
* RoleBinding
* Least-privilege access
* Workload identity

A useful mental model is:

```text
Workload
   ↓
ServiceAccount
   ↓
RBAC Permissions
   ↓
Allowed Kubernetes Resources
```

The important part is to keep the identity and permissions of a workload intentional rather than giving every workload unnecessary access.

---

# 🧹 Cleanup

Once the lab is complete, the entire namespace and the resources created inside it can be removed with:

```bash
kubectl delete ns dev1
```

This removes the `dev1` namespace along with the resources created within it.

---

#  Lab Summary

| Component               | Configuration |
| ----------------------- | ------------- |
| Namespace               | `dev1`        |
| ServiceAccount          | `demo-sa`     |
| Pod                     | `demo`        |
| Container Image         | `nginx`       |
| ServiceAccount assigned | `demo-sa`     |

### Final relationship

```text
dev1 Namespace
      │
      ├── demo-sa
      │
      └── demo Pod
             │
             └── ServiceAccount: demo-sa
```

This lab provides the basic foundation for understanding how Kubernetes workload identity connects with **RBAC and least-privilege security**.
