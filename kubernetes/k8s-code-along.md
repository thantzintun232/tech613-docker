# Kubernetes Code-Along (Nginx)

---

## 1. Deploy with YAML files

The app was defined in two YAML files:

| File | Creates |
|---|---|
| `nginx-deploy.yml` | Deployment, ReplicaSet and pods (3 replicas) |
| `nginx-service.yml` | Service that exposes the pods |

```bash
kubectl apply -f nginx-deploy.yml
kubectl apply -f nginx-service.yml
```

`apply` compares the file with what is running. If something is different, Kubernetes changes the cluster to match the file.

---

## 2. Check what is running

```bash
kubectl get all            # overview of everything
kubectl get pods           # pods only
kubectl get services       # services only
kubectl get rs             # replica sets only
kubectl get deployments    # deployments only
```

You don't need `get all` every time. Be specific to see only what you need.

---

## 3. Self-healing: delete a pod

**Goal:** check that Kubernetes replaces a deleted pod.

```bash
kubectl get pods                       # copy a pod name
kubectl delete pod <pod-name>          # delete it
kubectl get pods                      # check again straight away
```

- `kubectl delete <pod-name>` alone does **not** work. You must say it is a pod: `delete pod <name>` or `delete pod/<name>`.
- **Result:** 3 pods are still running. The deleted pod is gone and a **new pod with a new name and a younger age** replaced it.
- **Why:** the deployment says `replicas: 3`, so Kubernetes always tries to keep 3 healthy pods. This is **self-healing**.

---

## 4. Scale replicas with zero downtime

There are 3 ways to change the number of replicas while the app keeps running.

### Method 1: Edit the live deployment

```bash
kubectl get deployments               # get the deployment name
kubectl edit deploy <deployment-name>
```

- Opens the live YAML in a text editor.
- Change `spec: replicas: 3` to `4`, save and close.
- Check with `kubectl get pods` (now 4 pods).

> **Tip:** the default editor is Vim, which can be hard to use. Switch to nano with:
> ```bash
> export KUBE_EDITOR="nano"
> ```

### Method 2: Edit the YAML file and apply it

1. Open `nginx-deploy.yml` and change `replicas` to `5`.
2. Apply the file again:

```bash
kubectl apply -f nginx-deploy.yml
```

3. Kubernetes sees the file no longer matches the cluster and updates it (now 5 pods).

### Method 3: `kubectl scale`

```bash
kubectl scale --replicas=6 deployment/<deployment-name>
```

- Fastest way. Result: 6 pods.

### Summary

| Method | Command | Changes your YAML file? |
|---|---|---|
| 1. Edit | `kubectl edit deploy <name>` | No, only the live config |
| 2. Apply | `kubectl apply -f file.yml` | Yes |
| 3. Scale | `kubectl scale --replicas=N deployment/<name>` | No, only the live config |

Methods 1 and 3 change the running cluster only. To keep the change long-term, update the YAML file too.

---

## 5. Image tags and updates

In the deployment YAML, the `image:` line sets the container image for the pods.

- **Do not use `latest`** (or reuse the same tag) if you want updates to work.
- Kubernetes only compares the **image name and tag** in the file. It does not check the registry for a newer image.
- If the tag is the same, `kubectl apply` sees **no change** and does nothing.

**Correct way to update:**
1. Build and push the new image with a **new tag** (e.g. `v1` → `v2`).
2. Change the tag in the deployment YAML.
3. Run `kubectl apply -f <deploy-file>`.
4. Kubernetes does a **rolling update**: it replaces the pods one by one, so the app stays up.

---

## 6. Clean up: delete everything

```bash
kubectl delete -f nginx-service.yml    # service first
kubectl delete -f nginx-deploy.yml     # then deployment, replica set and pods
kubectl get all                        # only the default cluster service remains
```

- `delete -f <file>` removes everything defined in that file.
- **Delete the service first.** Users are cut off cleanly, instead of reaching pods that are being shut down and seeing strange errors. Always think about the **user experience**.

---

## Command Cheat Sheet

| Task | Command |
|---|---|
| Apply a YAML file | `kubectl apply -f <file>` |
| See everything | `kubectl get all` |
| See pods / services / rs / deployments | `kubectl get pods` / `services` / `rs` / `deployments` |
| Delete a pod | `kubectl delete pod <pod-name>` |
| Edit a live deployment | `kubectl edit deploy <name>` |
| Scale replicas | `kubectl scale --replicas=N deployment/<name>` |
| Delete using a file | `kubectl delete -f <file>` |
| Use nano as editor | `export KUBE_EDITOR="nano"` |

## Key Lessons

- Kubernetes keeps the **desired state** (e.g. 3 replicas) and fixes anything that doesn't match.
- Pods are **ephemeral**: deleted pods are replaced with new ones.
- Scaling works with **no downtime**.
- Always use **versioned image tags** so rolling updates work.
- Delete the **service first** when cleaning up.
