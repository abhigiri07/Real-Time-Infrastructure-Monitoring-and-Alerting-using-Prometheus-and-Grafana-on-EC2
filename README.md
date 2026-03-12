## Real-Time Infrastructure Monitoring and Alerting using Prometheus & Grafana on AWS EC2

## Project Overview

This project implements a **real-time infrastructure monitoring system** using **Prometheus, Grafana, and Node Exporter** on AWS EC2 instances.

The goal is to monitor **server health metrics such as CPU, Memory, and Disk usage** and visualize them through dashboards while triggering alerts when thresholds exceed predefined limits.

In traditional setups, administrators manually log into servers via SSH to check system health. This project automates that process by implementing a **centralized monitoring stack** that continuously collects metrics and displays them in real-time dashboards.

This solution helps **Site Reliability Engineers (SREs)** detect system issues proactively before they cause downtime.

---

## Architecture

The monitoring architecture consists of a **central monitoring server** and **multiple application servers**.

Monitoring Server:

* Prometheus (metrics collection)
* Grafana (visualization dashboards)

Application Servers:

* Node Exporter (system metrics exporter)

Workflow:

Application Servers → Node Exporter → Prometheus → Grafana Dashboards → Alerts

Prometheus periodically scrapes metrics from Node Exporter running on each server and stores them in its time-series database. Grafana then queries Prometheus to visualize the data.

---

## Technologies Used

| Technology     | Purpose                         |
| -------------- | ------------------------------- |
| AWS EC2        | Infrastructure hosting          |
| Prometheus     | Metrics collection & monitoring |
| Grafana        | Dashboard visualization         |
| Node Exporter  | System metrics exporter         |
| Linux (Ubuntu) | Server operating system         |

---

## Infrastructure Setup

This project uses **3 EC2 instances**.

| Instance | Role                                     |
| -------- | ---------------------------------------- |
| EC2-1    | Monitoring Server (Prometheus + Grafana) |
| EC2-2    | Application Server 1 (Node Exporter)     |
| EC2-3    | Application Server 2 (Node Exporter)     |

Recommended configuration:

* AMI: Ubuntu 22.04
* Instance Type: t2.micro
* Storage: 10 GB

---

## Security Group Configuration

The following ports must be open:

| Port | Description   |
| ---- | ------------- |
| 22   | SSH access    |
| 9090 | Prometheus UI |
| 3000 | Grafana UI    |
| 9100 | Node Exporter |

---

## Step 1: Install Node Exporter on Application Servers

SSH into each application server:

```
ssh ubuntu@<server-ip>
```

Download Node Exporter:

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
```

Extract the archive:

```
tar -xvf node_exporter-1.8.1.linux-amd64.tar.gz
cd node_exporter-1.8.1.linux-amd64
```

Start Node Exporter:

```
./node_exporter
```

Verify metrics endpoint:

```
http://<server-ip>:9100/metrics
```

---

## Step 2: Install Prometheus on Monitoring Server

Login to the monitoring server and download Prometheus.

```
wget https://github.com/prometheus/prometheus/releases/download/v2.53.0/prometheus-2.53.0.linux-amd64.tar.gz
```

Extract the file:

```
tar -xvf prometheus-2.53.0.linux-amd64.tar.gz
cd prometheus-2.53.0.linux-amd64
```

Start Prometheus:

```
./prometheus --config.file=prometheus.yml
```

Prometheus will run on:

```
http://<monitoring-server-ip>:9090
```

---

## Step 3: Configure Prometheus Targets

Edit the `prometheus.yml` file to add Node Exporter targets.

Example configuration:

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "node_exporters"

    static_configs:
      - targets:
        - "APP_SERVER_1_IP:9100"
        - "APP_SERVER_2_IP:9100"
```

Restart Prometheus after updating the configuration.

Verify targets:

```
http://<prometheus-ip>:9090/targets
```

The targets should display **UP status**.

---

## Step 4: Install Grafana

Install Grafana on the monitoring server.

```
sudo apt update
sudo apt install -y grafana
```

Start Grafana service:

```
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

Access Grafana:

```
http://<server-ip>:3000
```

Default login:

```
Username: admin
Password: admin
```

---

## Step 5: Connect Grafana to Prometheus

In Grafana:

1. Go to **Settings**
2. Click **Data Sources**
3. Select **Prometheus**

Prometheus URL:

```
http://localhost:9090
```

Click **Save & Test**.

---

## Step 6: Import Grafana Dashboard

Grafana provides a pre-built dashboard for Node Exporter.

Import Dashboard ID:

```
1860
```

This dashboard displays:

* CPU Usage
* Memory Usage
* Disk Usage
* Network Activity
* System Load

---

## Step 7: Create Custom Dashboard Panels

### CPU Usage Query

```
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

### Memory Utilization

```
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
```

### Disk Usage

```
(node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes * 100
```

These queries allow Grafana to display **real-time infrastructure metrics**.

---

## Step 8: Configure Alert Rules

Create an alert rule file named:

```
alerts.yml
```

Example alert configuration:

```
groups:
- name: node_alerts

  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 70
    for: 1m

    labels:
      severity: warning

    annotations:
      summary: "High CPU Usage Detected"
      description: "CPU usage above 70% for more than 1 minute"
```

Add this file to `prometheus.yml`:

```
rule_files:
  - "alerts.yml"
```

Restart Prometheus.

---

## Step 9: Trigger Test Alert

Install stress tool on application server:

```
sudo apt install stress
```

Generate CPU load:

```
stress --cpu 4
```

Check alert status:

```
http://<prometheus-ip>:9090/alerts
```

The alert should show **FIRING**.

---

## Monitoring Pipeline

Complete monitoring flow:
```
Application Servers
↓
Node Exporter
↓
Prometheus (scrapes metrics every 15s)
↓
Grafana Dashboards
↓
Alert Rules Trigger Notifications
```
---

## Project Deliverables

This repository includes:

* Prometheus configuration file
* Alert rules configuration
* Grafana dashboard screenshots
* Monitoring architecture documentation
---

