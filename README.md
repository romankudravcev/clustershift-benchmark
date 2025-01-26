# Benchmark Setup and Reproduction Guide

This guide explains the steps to set up and reproduce the benchmark for Kubernetes cluster migration.

---

## VM Specifications
- **Operating System**: Ubuntu 20.04
- **vCPUs**: 2
- **Memory**: 4 GB
- **Storage**: 30 GB SSD
- **Load Balancer**: MetalLB
- **Kubernetes**: k3s

Both VMs are hosted on a single physical on-premises machine and configured as two single-node clusters:
1. **Origin Cluster**: Source for migration
2. **Target Cluster**: Destination for migration

---

## Steps to Reproduce the Benchmark

### 1. Ensure Cluster Communication
- Verify that both clusters can communicate with each other.
- Adapt the UFW rules to allow the necessary traffic before starting the migration.

---

### 2. Deploy the Kubernetes Metrics Collector
- Install the metrics collector on both clusters.
- This tool monitors CPU and memory usage and writes the data to an SQLite database for post-benchmark analysis.

---

### 3. Install the CNPG Operator
- On the **origin cluster**, install the CloudNativePG (CNPG) operator.
- Wait until the operator is running.
```bash
kubectl apply --server-side -f \
  https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/release-1.25/releases/cnpg-1.25.0.yaml
```
---

### 4. Deploy the PostgreSQL Database
- Deploy a PostgreSQL cluster configuration on the **origin cluster**.
- Ensure that the database cluster becomes healthy.
```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: my-postgres-cluster
  namespace: postgres
spec:
  instances: 3
  primaryUpdateStrategy: unsupervised
  storage:
    size: 2Gi
    storageClass: local-path
```

---

### 5. Deploy the ClusterShift Benchmarker
- Deploy the benchmarker tool on the **origin cluster**.
- Retrieve the database secret and update the configuration file with the necessary values.

---

### 6. Simulate Requests
- Use the benchmarking client to simulate requests.
- Modify the configuration file as required and start the simulation.
```bash
go run .
```

---

### 7. Start the Migration
- Begin the migration process using the ClusterShift tool.
- Fill in the prompts or use default values as needed.
```bash
go run cmd/main.go migrate --origin kubeconfig-origin.yml --target kubeconfig-target.yml
```

---

### 8. Collect Benchmark Results
- Wait for the migration to complete.
- Use the REST APIs of the benchmark tools to collect data such as:
  - **Database Status**: Compare with POST requests sent during simulation to verify availability.
  - **CPU and Memory Usage**: Retrieve metrics from the metrics collector.
  - **Response Times**: Extract data from the benchmark client.

---

## Notes
- Ensure all tools and configurations are deployed and functioning properly before starting the migration.
- Use the collected data to analyze system performance and the impact of migration on database availability.
