# C8DashboardGrafana

This dashboard provides complete visibility over your Camunda 8 and Zeebe environments — monitoring process execution, job performance, incidents, Elasticsearch status, and Kubernetes resource utilization.

# 📊 Camunda 8 Monitoring Dashboard (`grafana.json`)

## 🧭 Overview
This dashboard provides a complete view of your **Camunda 8 / Zeebe** environment.  
It monitors process executions, job lifecycle, incidents, Elasticsearch health, and Kubernetes resource usage, offering full operational visibility.

---

## 🔹 Dashboard Sections

### **1. KPI Overview**
- Displays:
  - **Process instances**
  - **Flow nodes**
  - **Service tasks**
- Based on the metric:  
  ```promql
  increase(zeebe_element_instance_events_total[$KPI_TIME])
  ```
- Helps track workflow throughput and identify execution trends.

---

### **2. Zeebe Jobs**
- Visualizes **activated**, **completed**, and **canceled** jobs grouped by type.
- Uses:
  ```promql
  zeebe_job_events_total
  ```
- Useful for identifying job-type bottlenecks or stuck workers.

---

### **3. Incidents**
- Shows created vs. resolved incidents:
  ```promql
  zeebe_incidents_events_total
  ```
- A key indicator of system stability and recovery efficiency.

---

### **4. Elasticsearch**
- Tracks:
  - **Cluster health** (Online / Yellow / Red)
  - **Pod uptime**
  - **Index size and document count** for Zeebe, Operate, Tasklist, and Optimize.
- Helps detect index growth or unhealthy cluster states.

---

### **5. Resource Utilization**
- Displays **CPU and memory usage** for Zeebe, Gateway, and Operate pods.
- Metrics used:
  ```promql
  container_cpu_usage_seconds_total
  container_memory_working_set_bytes
  ```
  compared against  
  ```promql
  kube_pod_container_resource_limits
  ```
- Ensures performance and scaling consistency in Kubernetes deployments.

---

## ⚙️ Datasource & Variables

### **Datasource**
- Prometheus (`$DS_PROMETHEUS`)
- Must expose:
  - Zeebe metrics
  - Elasticsearch metrics
  - Kubernetes node/container metrics

### **Variables**
| Variable | Description | Example |
|-----------|-------------|----------|
| `$namespace` | Regex filter for environment selection | `prod|hml|uat` |
| `$KPI_TIME` | Time interval for metrics aggregation | `1h, 6h, 12h, 24h, 7d` |

---

## 🚀 Import & Setup Steps

1. **Import the Dashboard**
   - Navigate to **Grafana → Dashboards → Import**.
   - Upload `grafana.json`.

2. **Select Datasource**
   - Choose **Prometheus** as the data source.

3. **Configure Variables**
   - Go to **Settings → Variables**:
     - `namespace`: *Query / Regex* type for environment labels.
     - `KPI_TIME`: *Custom / Interval* type for time options.

4. **Validate Prometheus Labels**
   - Ensure metrics use consistent labels such as `container`, `pod`, `index`, and `partition`.

5. **Test Visualization**
   - Open the dashboard, select your namespace and KPI time window, and verify data population.

---

## ⚠️ Recommended Alerts

| Area | Condition | Expression / Trigger |
|------|------------|----------------------|
| **Elasticsearch Health** | Cluster not “Online” for >5 min | Status ≠ `Online` |
| **Incident Spike** | New incidents in 5 min | `increase(zeebe_incidents_events_total{action="created"}[5m]) > 0` |
| **High Resource Usage** | CPU or memory > 80% for 10 min | Compare `%usage` vs. limit |
| **Write Stop Detection** | RocksDB write stop triggered | `zeebe_rocksdb_writes_is_write_stopped == 1` |

---

## 💡 Suggested Improvements

1. **Unified Jobs Panel**  
   Merge all job-related panels (activated, completed, canceled) into a single stacked bar chart by `action` and `type`.

2. **Job Latency Tracking**  
   Add p95/p99 latency visualization:
   ```promql
   histogram_quantile(0.95, rate(zeebe_job_duration_seconds_bucket[5m]))
   ```

3. **Completion Rate Metric**  
   Show job success ratio:
   ```promql
   completed / (activated + completed + canceled)
   ```

4. **Capacity vs. Demand**  
   Include `zeebe_gateway_job_stream_*` metrics and worker counts to monitor job consumption rate.

5. **Annotations for Deployments**  
   Use Grafana annotations to mark releases and maintenance windows (already supported in JSON).

