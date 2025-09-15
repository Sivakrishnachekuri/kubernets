# Kubernetes Resource Quota — Complete Guide with Examples

---

## Concept:
ResourceQuota is used to limit the total amount of resources that a **Namespace** can use in Kubernetes.

---

## Why We Use Resource Quota:
- Prevent one team or app from consuming **all cluster resources**.
- Provide **fair resource distribution** across teams or projects.
- Control CPU, Memory, Storage, and Object counts (like Pods, Services).

---

## Resource Quota Applies To:
- CPU  
- Memory  
- Storage  
- Object Counts (Pods, Services, ConfigMaps, etc.)  

---

## Key Resource Types:
| Resource Type         | Meaning                                           |
|-----------------------|---------------------------------------------------|
| requests.cpu          | Total CPU requested across all Pods               |
| requests.memory       | Total memory requested across all Pods            |
| limits.cpu            | Total CPU limit across all Pods                   |
| limits.memory         | Total memory limit across all Pods                |
| pods                  | Total number of Pods allowed                      |
| configmaps            | Maximum number of ConfigMaps allowed              |
| persistentvolumeclaims| Total PVCs allowed                                |

---

## Step-by-Step Example 1: Limit CPU & Memory in Namespace

### 1. Create Namespace:
```bash
kubectl create namespace dev-team
````

---

### 2. Define ResourceQuota YAML:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-team-quota
  namespace: dev-team
spec:
  hard:
    requests.cpu: "2"
    requests.memory: "4Gi"
    limits.cpu: "3"
    limits.memory: "6Gi"
```

---

### Meaning:

* Maximum total CPU requests allowed: **2 cores**.
* Maximum total Memory requests allowed: **4Gi**.
* Total CPU limits: **3 cores**.
* Total Memory limits: **6Gi**.

---

### 3. Apply Quota:

```bash
kubectl apply -f quota.yaml
```

---

### 4. Verify:

```bash
kubectl get resourcequota -n dev-team
```

---

## Step-by-Step Example 2: Limit Number of Pods & ConfigMaps

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-count-quota
  namespace: dev-team
spec:
  hard:
    pods: "10"
    configmaps: "20"
    persistentvolumeclaims: "5"
```

---

### Meaning:

* Max 10 Pods allowed.
* Max 20 ConfigMaps allowed.
* Max 5 PersistentVolumeClaims allowed.

---

## Step-by-Step Example 3: Storage Resource Quota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-quota
  namespace: dev-team
spec:
  hard:
    requests.storage: "100Gi"
```

---

### Meaning:

* Total storage requests across PVCs cannot exceed **100Gi**.

---

## Important Notes:

* ResourceQuotas apply **within a Namespace**.
* They are **enforced automatically** by Kubernetes.
* If exceeded, Pod creation or resource allocation fails with error:

```
exceeded quota
```

---

## Command Summary:

| Task                   | Command Example                                         |
| ---------------------- | ------------------------------------------------------- |
| Create Namespace       | kubectl create namespace dev-team                       |
| Apply ResourceQuota    | kubectl apply -f quota.yaml                             |
| View ResourceQuota     | kubectl get resourcequota -n dev-team                   |
| Describe ResourceQuota | kubectl describe resourcequota -n dev-team              |
| Delete ResourceQuota   | kubectl delete resourcequota dev-team-quota -n dev-team |

---

## Troubleshooting Tips:

* Always apply ResourceQuota **after Namespace exists**.
* If Pod creation fails, check:

```bash
kubectl describe resourcequota -n <namespace>
```

* Ensure Pods specify **requests and limits** properly:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "1Gi"
  limits:
    cpu: "1"
    memory: "2Gi"
```

---

## Common Use Cases:

| Scenario                         | Solution                                      |
| -------------------------------- | --------------------------------------------- |
| Limit CPU & Memory Usage         | Define ResourceQuota with CPU & Memory fields |
| Control Number of Objects (Pods) | Use object-count quotas                       |
| Restrict PVC Storage Consumption | Set `requests.storage` limits                 |
| Isolate Teams by Namespaces      | Assign separate quotas per team Namespace     |

---

.

