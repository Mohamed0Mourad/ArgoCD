
# ArgoCD Deployment Examples

This repository demonstrates different ways to deploy applications to [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) using:

- Raw Kubernetes **Manifest files**
- **Helm Charts**
- **Kustomize**

> 💡 Learn how to manage app deployments using GitOps with practical examples.

## 🚀 Getting Started With Manifest files

### 1. Add the Repository to ArgoCD

```bash
argocd repo add "https://github.com/Mohamed0Mourad/ArgoCD.git" \
  --username "[your username]" \
  --password "[your personal token]"
```

---

### 2. Create a Directory for ArgoCD Setup

```bash
mkdir argo-cd
cd argo-cd
```

---

### 3. Create the ArgoCD Application YAML

Save the following file as `mynginxapp.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/Mohamed0Mourad/ArgoCD.git'
    path: manifests
    targetRevision: main
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

Apply it to the cluster:

```bash
kubectl apply -f mynginxapp.yaml
```

---

### 4. View in ArgoCD Dashboard

Open the ArgoCD UI → Applications → You should see the `nginx` application synced and healthy.

---

### 5. Check Kubernetes Resources

```bash
kubectl get pods
kubectl get services
```

**Example Output:**
```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-5d568c4c64-k2m4q   1/1     Running   0          3m
nginx-5d568c4c64-nrbxk   1/1     Running   0          3m

NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nginx-svc    ClusterIP   10.96.86.130   <none>        80/TCP    3m
```

---

## 🔄 Updating the Application (e.g., Scaling Replicas)

Let’s scale the replicas from **2 to 6**.

### 1. Create a Feature Branch

```bash
git checkout -b feature/increase-replicas
```

### 2. Edit `manifests/deployment.yml`

Change:
```yaml
replicas: 2
```

To:
```yaml
replicas: 6
```

### 3. Commit and Push

```bash
git add manifests/deployment.yml
git commit -m "Increase the replicas"
git push --set-upstream origin feature/increase-replicas
```

Go to GitHub, open the pull request, and approve the merge.

---

### 4. Sync the App (Optional)

ArgoCD automatically syncs every 3 minutes, but you can trigger it manually:

```bash
argocd app sync nginx
```

Then verify:

```bash
kubectl get pods
```

**Expected Output:**
```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-5d568c4c64-abcd1   1/1     Running   0          1m
nginx-5d568c4c64-abcd2   1/1     Running   0          1m
nginx-5d568c4c64-abcd3   1/1     Running   0          1m
nginx-5d568c4c64-abcd4   1/1     Running   0          1m
nginx-5d568c4c64-abcd5   1/1     Running   0          1m
nginx-5d568c4c64-abcd6   1/1     Running   0          1m
```

---

### ✅ Final Result

Below is a screenshot after scaling to 6 pods in the ArgoCD UI:
📸 
![Screenshot from 2025-04-16 00-41-29](https://github.com/user-attachments/assets/78803b2d-be0c-4a79-ba86-6ec390837ca3)


## 2️⃣ Deploying Application Using a Community Helm Chart

This section demonstrates how to deploy an application using a public Helm chart (`httpbin`).

---

### Create `httpbin.yaml`

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: httpbin
  namespace: argocd
spec:
  project: default
  source:
    chart: httpbin
    repoURL: https://matheusfm.dev/charts
    targetRevision: 0.1.1
    helm:
      releaseName: httpbin
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

---

### Steps to Deploy

```bash
git checkout -b feature/add-httpbin-chart
git add httpbin.yaml
git commit -m "Adds the HTTPbin chart"
git push --set-upstream origin feature/add-httpbin-chart
```

1. Copy the merge request link that appears in the terminal.
2. Open it in your browser.
3. Approve and merge the MR.

---

### Refresh and Sync in ArgoCD

- Go to the ArgoCD UI
- Click **Refresh** to fetch the new application
- You will see the **HTTPbin** app with a Helm chart icon

---

### Verify Deployment

```bash
kubectl get pods
```

**Example Output:**

```
NAME                       READY   STATUS    RESTARTS   AGE
httpbin-xxxxxx-yyyyy       1/1     Running   0          1m
```

Your HTTPbin application is now running in the cluster via Helm and fully GitOps-enabled with ArgoCD.

![Screenshot from 2025-04-16 02-50-51](https://github.com/user-attachments/assets/6f1770c7-e3f0-4713-bf7c-8eb9a60fa0f7)
