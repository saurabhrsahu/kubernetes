# Kubernetes demo setups

This file is **README.md** so GitHub and editors can render it as Markdown (use Markdown preview in Cursor/VS Code if needed).

This repository holds **hands-on, demo-oriented** Kubernetes manifests: a **guided sample app** plus **standalone database stacks** you can apply on a dev cluster. Everything is meant for **learning and local/testing clusters**—change all default passwords before any real environment.

## Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) configured against a cluster
- A default **StorageClass** (for PersistentVolumes), unless you set `storageClassName` on PVCs / StatefulSet volume claim templates

## Contents

| Path | What it is |
|------|------------|
| [**GUIDE.md**](GUIDE.md) | How to use Kubernetes with this repo: concepts, applying manifests, verifying workloads, optional components |
| [**samples/**](samples/) | End-to-end **walkthrough**: namespace, quotas, ConfigMap/Secret, RBAC, Pod, Deployment, Service, Ingress, HPA, PDB, NetworkPolicy, PVC—with comments in YAML |
| [**mysql/**](mysql/) | MySQL 8: Namespace, Secret, PVC, Deployment, Service (`kubectl apply -k .`) — see [mysql/APPLY.txt](mysql/APPLY.txt) |
| [**postgres/**](postgres/) | PostgreSQL 16 (Alpine): same pattern — [postgres/APPLY.txt](postgres/APPLY.txt) |
| [**mongo/**](mongo/) | MongoDB 7: root user + DB via `MONGO_INITDB_*` — [mongo/APPLY.txt](mongo/APPLY.txt) |
| [**cassandra/**](cassandra/) | Cassandra 5: **StatefulSet** + headless service for seeds, ConfigMap for cluster settings — [cassandra/APPLY.txt](cassandra/APPLY.txt) |
| [**redis/**](redis/) | Redis 7 (Alpine): password + AOF on PVC — [redis/APPLY.txt](redis/APPLY.txt) |
| [**dynamodb/**](dynamodb/) | DynamoDB **Local** (official image): API-compatible dev/test — [dynamodb/APPLY.txt](dynamodb/APPLY.txt) |

## Quick start (each stack)

From the folder you want:

```bash
cd samples    # or mysql, postgres, mongo, cassandra, redis, dynamodb
kubectl apply -k .
```

Dry-run merged output:

```bash
kubectl kustomize .
```

Check resources (replace namespace as needed):

```bash
kubectl -n demo-app get pods    # samples use namespace demo-app
kubectl -n mysql get pods
```

Tear down a database demo in one shot:

```bash
kubectl delete namespace mysql   # or postgres, mongo, cassandra, redis, dynamodb
```

For **samples**, delete `namespace demo-app` (see [samples/APPLY.txt](samples/APPLY.txt)).

## Notes

- **Secrets** in this repo use placeholder values. Treat them as **examples only**.
- **Ingress, HPA, and NetworkPolicy** in `samples/` need a matching controller, metrics-server, and CNI behavior—see [GUIDE.md](GUIDE.md).
- **Cassandra** uses a StatefulSet (not a Deployment) so pod DNS and seeds stay stable.
- **MySQL** defines both a root password and an app user; **Postgres** uses one superuser in this sample; **Mongo** uses init root credentials; **Redis** uses `requirepass`; **Cassandra** ships without CQL auth in this demo.
- **DynamoDB** here is [DynamoDB Local](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html) in-cluster—not AWS-hosted DynamoDB; use it for API-compatible local testing.

For a full narrative and command reference, start with **[GUIDE.md](GUIDE.md)**.
