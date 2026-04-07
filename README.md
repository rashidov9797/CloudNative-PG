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
