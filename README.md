# CloudNative-PG
Cloud-Native PostgreSQL 18 HA Cluster (Kubernetes)


# 🚀 PostgreSQL 18 HA Cluster on Kubernetes (CloudNativePG)

## 📋 Project Overview

This project demonstrates a **production-grade PostgreSQL High Availability platform** built on Kubernetes (K3s) using CloudNativePG.

Key features:

- 1 Primary + 8 Standby replicas
- Streaming replication
- PgBouncer connection pooling (RW / RO)
- S3-compatible backup (MinIO)
- Prometheus + Grafana monitoring
- Load testing with pgbench
- Automatic failover validation

---

## 🏗️ Architecture

<img width="631" height="502" alt="image" src="https://github.com/user-attachments/assets/333f7eb6-c9be-47f3-923b-fea1146db9db" />


---

## 🖥️ Environment & Versions

- OS: Rocky Linux 8
- Kubernetes: K3s
- PostgreSQL: 18
- Operator: CloudNativePG
- Pooling: PgBouncer (CNPG Pooler)
- Backup: MinIO (S3)
- Monitoring: Prometheus + Grafana

---

## ⚙️ Key Parameters

- Instances: 9 (1 primary + 8 standby)
- Storage: 10Gi
- Pool mode: transaction
- Backup: S3 (MinIO)
- Replication: streaming

---

## 🔄 HA & Replication Status

```bash
kubectl exec -it pg-demo-1 -n cnpg-cluster -- \
psql -U postgres -c "select application_name,state,sync_state from pg_stat_replication;"

```

<img width="983" height="486" alt="image" src="https://github.com/user-attachments/assets/a339b6d2-98f6-4e35-bd87-14dacf2a73db" />



## 📦 Cluster Pod Status

``` bash
kubectl get pods -n cnpg-cluster
```

<img width="784" height="391" alt="image" src="https://github.com/user-attachments/assets/4c3eeaac-cb75-4eb9-8133-7ae9e1f40df5" />



## 💾 Backup Validation

``` bash
kubectl get backup -n cnpg-cluster
```

<img width="989" height="324" alt="image" src="https://github.com/user-attachments/assets/261cc46d-d493-496c-a306-eb98e871c109" />

✔ Base backup completed
✔ WAL archive active




## 📈 Read-Only Load Test via PgBouncer

To validate read traffic through the read-only pooler, a pgbench workload was generated from a client pod inside the Kubernetes cluster.



```bash
kubectl exec -it pgbench -n cnpg-cluster -- \
  bash -c "PGPASSWORD=$PGPASS pgbench -h pooler-ro -p 5432 -U appuser -d appdb -S -n -c 600 -j 16 -T 300"

```


## 📊 Standby Connection Distribution

To observe how connections are distributed across standby nodes during the load test:

### Command

```bash
while true; do
  date '+%F %T'
  for pod in pg-demo-2 pg-demo-3 pg-demo-4 pg-demo-5 pg-demo-6 pg-demo-7 pg-demo-8 pg-demo-9; do
    CONN=$(kubectl exec -t $pod -n cnpg-cluster -c postgres -- \
      psql -U postgres -t -A -c "SELECT count(*) FROM pg_stat_activity WHERE usename='appuser';" | tr -d '\r')
    echo "$pod -> total_connections=$CONN"
  done
  echo "-----------------------------"
  sleep 2
done

```

Sample Output

```
2026-04-07 17:04:41
pg-demo-2 -> total_connections=53
pg-demo-3 -> total_connections=73
pg-demo-4 -> total_connections=60
pg-demo-5 -> total_connections=85
pg-demo-6 -> total_connections=92
pg-demo-7 -> total_connections=46
pg-demo-8 -> total_connections=65
pg-demo-9 -> total_connections=71
```

Result
- Read traffic was distributed across all standby nodes
- Each standby handled active sessions
- Load was balanced across replicas

<img width="1152" height="772" alt="image" src="https://github.com/user-attachments/assets/69cf19ce-d845-4bc5-8385-2fb7aff976cb" />



## 📊 Monitoring (Grafana)

Grafana was used to visualize PostgreSQL cluster performance and behavior during load testing.

### Dashboard Overview

<img width="1903" height="920" alt="image" src="https://github.com/user-attachments/assets/efe08c18-ace1-4540-a40b-3d013b6b3f25" />

- - - 


<img width="1888" height="911" alt="image" src="https://github.com/user-attachments/assets/656bd9ce-f0c6-4276-8ad0-55765d774281" />


### Observed Metrics

- **Connections**
  - Active connections increased during pgbench load
  - Connections were distributed across standby nodes

- **Transactions / TPS**
  - Visible spike during load test execution
  - System handled concurrent read workload

- **Replication Lag**
  - Stayed close to `0`
  - Indicates healthy streaming replication

- **CPU / Memory Usage**
  - Increased under load
  - Remained stable without crashes

### Key Observations

- Read traffic was successfully handled by standby replicas
- No single node was overloaded
- Cluster remained stable during high concurrency
- Monitoring confirmed system behavior in real time

### Purpose

- Validate system performance under load
- Observe replication health
- Ensure cluster stability during traffic spikes





# 🔥 Failover Test (Primary Node Failure)

To verify high availability, the primary PostgreSQL node was manually deleted to simulate a failure.

### Step 1: Delete Primary Pod

```bash
kubectl delete pod pg-demo-1 -n cnpg-cluster --force --grace-period=0
```

Step 2: Observe Cluster Behavior

```bash
kubectl get pods -n cnpg-cluster -w
```

- Original primary pod is terminated
- A standby node is automatically promoted to primary
- Remaining replicas reconnect to the new primary

<img width="1157" height="517" alt="image" src="https://github.com/user-attachments/assets/98067b31-5f59-42c2-a441-593af012776c" />


- - - 

Step 3: Identify New Primary

``` bash
kubectl get cluster pg-demo -n cnpg-cluster -o jsonpath='{.status.currentPrimary}'; echo
```

<img width="1253" height="352" alt="image" src="https://github.com/user-attachments/assets/1fea1ef3-5e7e-4fd2-901d-41baf893bd31" />


Grafana Dashboard

<img width="1725" height="782" alt="image" src="https://github.com/user-attachments/assets/dd7238bf-b1a2-48c7-9fc0-2c0141cb8908" />

