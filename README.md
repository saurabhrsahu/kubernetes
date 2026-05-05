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
| [**mariadb/**](mariadb/) | MariaDB LTS: same pattern as MySQL; `MARIADB_*` env from Secret — [mariadb/APPLY.txt](mariadb/APPLY.txt) |
| [**postgres/**](postgres/) | PostgreSQL 16 (Alpine): same pattern — [postgres/APPLY.txt](postgres/APPLY.txt) |
| [**mongo/**](mongo/) | MongoDB 7: root user + DB via `MONGO_INITDB_*` — [mongo/APPLY.txt](mongo/APPLY.txt) |
| [**cassandra/**](cassandra/) | Cassandra 5: **StatefulSet** + headless service for seeds, ConfigMap for cluster settings — [cassandra/APPLY.txt](cassandra/APPLY.txt) |
| [**redis/**](redis/) | Redis 7 (Alpine): password + AOF on PVC — [redis/APPLY.txt](redis/APPLY.txt) |
| [**dynamodb/**](dynamodb/) | DynamoDB **Local** (official image): API-compatible dev/test — [dynamodb/APPLY.txt](dynamodb/APPLY.txt) |
| [**cockroach/**](cockroach/) | CockroachDB single-node (`start-single-node`, insecure demo): SQL + DB Console — [cockroach/APPLY.txt](cockroach/APPLY.txt) |
| [**influx/**](influx/) | InfluxDB **v2** (`2.8-alpine`): UI + API on 8086, `DOCKER_INFLUXDB_INIT_*` setup — [influx/APPLY.txt](influx/APPLY.txt) |
| [**prometheus/**](prometheus/) | Prometheus **v2**: ConfigMap + PVC for TSDB, namespace-local Pod RBAC + scrape — [prometheus/APPLY.txt](prometheus/APPLY.txt) |
| [**oracledb/**](oracledb/) | Oracle Database **Free** (`gvenzl/oracle-free`): listener 1521, PDB **FREEPDB1** — [oracledb/APPLY.txt](oracledb/APPLY.txt) |
| [**couchdb/**](couchdb/) | Apache CouchDB **3** (official image): HTTP API + Fauxton on **5984**, admin via env — [couchdb/APPLY.txt](couchdb/APPLY.txt) |

## Quick start (each stack)

From the folder you want:

```bash
cd samples    # or mysql, mariadb, postgres, mongo, cassandra, redis, dynamodb, cockroach, influx, prometheus, oracledb, couchdb
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
kubectl delete namespace mysql   # or mariadb, postgres, mongo, cassandra, redis, dynamodb, cockroach, influx, prometheus, oracledb, couchdb
```

For **samples**, delete `namespace demo-app` (see [samples/APPLY.txt](samples/APPLY.txt)).

## Notes

- **Secrets** in this repo use placeholder values. Treat them as **examples only**.
- **Ingress, HPA, and NetworkPolicy** in `samples/` need a matching controller, metrics-server, and CNI behavior—see [GUIDE.md](GUIDE.md).
- **Cassandra** uses a StatefulSet (not a Deployment) so pod DNS and seeds stay stable.
- **MySQL** and **MariaDB** define root plus an app user/database via image env vars (different Secret keys / env prefixes; MariaDB uses `MARIADB_*`); **Postgres** uses one superuser in this sample; **Mongo** uses init root credentials; **Redis** uses `requirepass`; **Cassandra** ships without CQL auth in this demo.
- **DynamoDB** here is [DynamoDB Local](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html) in-cluster—not AWS-hosted DynamoDB; use it for API-compatible local testing.
- **CockroachDB** in `cockroach/` is a **single-node**, **`--insecure`** demo for local learning—use TLS and a supported production topology for real workloads.
- **InfluxDB** in `influx/` pins **v2** (`influxdb:2.8-alpine`); Docker Hub may move `latest` to **InfluxDB 3**—see [official image tags](https://hub.docker.com/_/influxdb).
- **Prometheus** in `prometheus/` ships a single replica with **namespace-scoped** discovery (`prometheus` namespace only); extend RBAC/config for full-cluster scraping or use the [Operator](https://prometheus-operator.dev/).
- **OracleDB** in `oracledb/` uses the community **[gvenzl/oracle-free](https://hub.docker.com/r/gvenzl/oracle-free)** image (Oracle Database Free); it is **heavy** on first boot and memory—see [oracledb/APPLY.txt](oracledb/APPLY.txt) and Oracle’s license terms.
- **CouchDB** in `couchdb/` is single-node; complete the **single-node** setup in [Fauxton](https://docs.couchdb.org/en/stable/install/setup.html#single-node-setup) (`/_utils`) once per fresh volume so system databases exist.

For a full narrative and command reference, start with **[GUIDE.md](GUIDE.md)**.
