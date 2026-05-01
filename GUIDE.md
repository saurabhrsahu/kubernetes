# Kubernetes: how to use it (with a full example)

This document explains **how to work with Kubernetes** in practice, using the **sample manifests in the `samples/` folder** as a single, end-to-end example app (`demo-app` namespace, a small web workload).

---

## 1. What you are actually doing

Kubernetes runs **declarative** workloads: you describe **desired state** in YAML, and the control plane **reconciles** running processes (Pods) and networking (Services, Ingress) to match.

| Idea | In one sentence |
|------|-----------------|
| **Cluster** | One or more nodes + control plane (API, scheduler, controllers). |
| **Pod** | Smallest deployable unit; one or more containers sharing network/storage. |
| **Deployment** | Keeps a desired number of Pod **replicas** and rolls out updates. |
| **Service** | Stable name + IP (and DNS) that **load-balances** to matching Pods. |
| **ConfigMap / Secret** | Config and sensitive values injected as env or files. |
| **Ingress** | HTTP/HTTPS rules from outside the cluster to Services (needs an **Ingress controller**). |
| **Namespace** | Isolation and naming scope for most objects. |

Most day-to-day work: **`kubectl` talks to the API**, you **apply** YAML, and you **observe** with `get`, `describe`, `logs`, `port-forward`, etc.

---

## 2. What you need installed

- **`kubectl`**: the CLI to the cluster API. [Install kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl).
- **A cluster** (any one of these is enough to learn):
  - [minikube](https://minikube.sigs.k8s.io/docs/start/)
  - [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
  - A cloud managed cluster (EKS, AKS, GKE, etc.)
  - [Docker Desktop](https://docs.docker.com/desktop/kubernetes/) (built-in Kubernetes)

Check:

```bash
kubectl version --client
kubectl cluster-info
kubectl get nodes
```

If `get nodes` works, you can use the rest of this guide.

---

## 3. How the example in `samples/` is structured

All numbered YAML files build a **toy but realistic** stack. Apply order is already encoded in `samples/kustomization.yaml`.

| File | What it is for |
|------|----------------|
| `01-namespace.yaml` | Creates namespace `demo-app`. |
| `02-resourcequota-limitrange.yaml` | Caps total resources in the namespace; default min/max for containers. |
| `03-configmap-secret.yaml` | Non-secret config and example Secret keys. |
| `04-serviceaccount-rbac.yaml` | Identity for the workload + Role/RoleBinding (what the app may read in the API). |
| `05-pod-standalone.yaml` | One-off Pod (educational; not the main way you run things). |
| `06-deployment.yaml` | **Main app**: 2x nginx Pods, env from ConfigMap/Secret, probes, resources. |
| `07-service.yaml` | **ClusterIP** Service `web` on port 80 → Pods with `app: web`. |
| `08-ingress.yaml` | HTTP route (needs **IngressClass** + controller, e.g. nginx). |
| `09-hpa.yaml` | Autoscaling 2–5 replicas (needs **metrics-server**). |
| `10-pdb.yaml` | At least one Pod available during voluntary disruptions. |
| `11-networkpolicy.yaml` | Tighten who can talk to the Pods (CNI must enforce policies). |
| `12-persistentvolumeclaim.yaml` | Example 1 Gi claim (not mounted in the Deployment by default). |

**How traffic fits together (happy path):**

- Outside cluster (browser) → **Ingress** (if installed) → **Service** `web:80` → **Endpoints** (Pod IPs) → **container** `nginx` (port 80).
- If you have no Ingress, you can still use **`kubectl port-forward`** to the Service (see section 5).

**Labels matter:** the Deployment template sets `app: web`. The Service `selector` is `app: web`. The standalone Pod uses `app: web-standalone` so the Service does **not** accidentally include it.

---

## 4. Two ways to apply everything

From the `samples` directory:

**Option A – Kustomize (recommended for this repo):**

```bash
cd samples
kubectl apply -k .
```

**Option B – File by file (same order as `kustomization.yaml`):**

```bash
cd samples
kubectl apply -f 01-namespace.yaml
kubectl apply -f 02-resourcequota-limitrange.yaml
# ... continue through 12, or:
kubectl apply -f .
```

**Dry-run (print merged YAML, no change to cluster):**

```bash
kubectl kustomize .
```

---

## 5. Verify the stack (commands you will use all the time)

```bash
# Is the namespace there?
kubectl get namespace demo-app

# Pods, Deployments, Services,Ingress
kubectl -n demo-app get pods,deploy,svc,ingress

# If something is not Ready, inspect events and spec
kubectl -n demo-app describe pod <pod-name>
kubectl -n demo-app get events --sort-by='.lastTimestamp'
```

**Reach the app without Ingress** (works on any cluster):

```bash
kubectl -n demo-app port-forward service/web 8080:80
# Open http://127.0.0.1:8080
```

**Logs:**

```bash
kubectl -n demo-app logs -l app=web -f
```

**Exec into a container (debugging only):**

```bash
kubectl -n demo-app exec -it deploy/web -- sh
```

If **Ingress (08)** is used, install an [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) first, then:

```bash
kubectl get ingress -n demo-app
```

Point `web.demo.local` to your node / load balancer, or change `host` in `08-ingress.yaml` to a hostname you control.

If **HPA (09)** stays in `Progressing` or shows no metrics, install [metrics-server](https://github.com/kubernetes-sigs/metrics-server) in the cluster, or remove `09-hpa.yaml` from `kustomization.yaml` and re-apply.

If **NetworkPolicy (11)** blocks health checks or DNS, your CNI and rules must match. Easiest for learning: comment out that resource in `kustomization.yaml` until the rest is green.

**PVC (12):** ensure a `StorageClass` exists (`kubectl get storageclass`) or set `storageClassName` in the PVC.

---

## 6. A minimal path vs “everything on”

| Goal | Suggestion |
|------|------------|
| Fastest “hello stack” | Apply `01` through `07` (namespace → deployment → service). Use `port-forward` to test. |
| With Ingress | Add an Ingress controller, then apply `08` and fix `ingressClassName` + `host`. |
| With autoscaling | Install metrics-server, then use `09`. |
| Hardening | Add `10` (PDB) and `11` (NetworkPolicy) when basics work. |
| Storage demo | Use `12` and optionally add a `volume` + `volumeMount` in `06-deployment.yaml`. |

To **skip the standalone Pod** (file `05`), remove `05-pod-standalone.yaml` from the `resources:` list in `kustomization.yaml`.

---

## 7. How to change the example safely

- **Image / replicas:** edit `06-deployment.yaml` (`spec.replicas`, `image`).
- **Config / secrets:** edit `03-configmap-secret.yaml`; Deployment picks up ConfigMap/Secret **changes to env from** only on Pod restart (rollout restart: `kubectl -n demo-app rollout restart deploy/web`).
- **Exposure:** `ClusterIP` (default) is internal; `NodePort` or `LoadBalancer` changes `07-service.yaml` for different cluster/network setups.
- **Cleanup:** `kubectl delete namespace demo-app` removes almost everything in one shot (including cluster-scoped references tied to that namespace for namespaced resources).

---

## 8. What to read next (official docs)

- [Kubernetes core concepts](https://kubernetes.io/docs/concepts/)
- [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [API reference](https://kubernetes.io/docs/reference/kubernetes-api/) (for exact fields per kind)

The YAML files in `samples/` also contain **field-level comments** on the important `spec` and `metadata` fields.

---

## 9. Quick reference: file → `kubectl` object

| `kubectl` short name | Full kind (example) |
|----------------------|------------------------|
| `ns` | Namespace |
| `cm` | ConfigMap |
| `secret` | Secret |
| `sa` | ServiceAccount |
| `deploy` | Deployment |
| `po` (pods) | Pod |
| `svc` | Service |
| `ing` | Ingress |
| `hpa` | HorizontalPodAutoscaler |
| `pdb` | PodDisruptionBudget |
| `netpol` | NetworkPolicy |
| `pvc` | PersistentVolumeClaim |

This guide and the `samples/` manifests are meant to be used together: **read the guide for flow, read the YAML for every-field detail in context.**
