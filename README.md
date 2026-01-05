# 📊 Task: Monitor EC2 CPU & Memory using Prometheus and Grafana

Objective:
Set up Prometheus to collect EC2 system metrics and visualize them using Grafana dashboards.

🧱 Architecture Flow (High Level)
EC2 Instance
 └── Node Exporter (metrics)
        ↓
   Prometheus (scrape)
        ↓
     Grafana (visualize)

✅ Prerequisites

EC2 instance (Amazon Linux 2 / Ubuntu)

Ports opened in Security Group:

9090 → Prometheus

9100 → Node Exporter

3000 → Grafana

SSH access to EC2

🔹 Step 1: Install Node Exporter (Metric Collector)

Node Exporter collects CPU, memory, disk, network metrics.

cd /opt
sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
sudo tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64 node_exporter

Start Node Exporter
cd /opt/node_exporter
sudo ./node_exporter &

Verify
curl http://localhost:9100/metrics


✅ You should see a long list of metrics.

🔹 Step 2: Install Prometheus
cd /opt
sudo wget https://github.com/prometheus/prometheus/releases/download/v2.50.0/prometheus-2.50.0.linux-amd64.tar.gz
sudo tar -xvf prometheus-2.50.0.linux-amd64.tar.gz
sudo mv prometheus-2.50.0.linux-amd64 prometheus

🔹 Step 3: Configure Prometheus

Edit configuration file:

cd /opt/prometheus
sudo nano prometheus.yml

Add this job configuration:
scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]


Save & exit.

🔹 Step 4: Start Prometheus
cd /opt/prometheus
sudo ./prometheus \
--config.file=prometheus.yml \
--storage.tsdb.path=data &

Verify

Open browser:

http://<EC2-PUBLIC-IP>:9090


Go to Status → Targets
✅ node_exporter should be UP

🔹 Step 5: Install Grafana
Add Grafana repo & install
sudo yum install -y https://dl.grafana.com/oss/release/grafana-10.2.3-1.x86_64.rpm
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

Access Grafana
http://<EC2-PUBLIC-IP>:3000


Login:

Username: admin

Password: admin

🔹 Step 6: Add Prometheus as Data Source in Grafana

Grafana → ⚙️ Settings

Data Sources

Add data source

Select Prometheus

URL:

http://localhost:9090


Click Save & Test

✅ Data source connected successfully

🔹 Step 7: Import Ready-Made Dashboard

Grafana → + → Import

Dashboard ID:

1860


Select Prometheus data source

Click Import

🎉 You’ll see:

CPU Usage

Memory Usage

Disk IO

Network Traffic

🔹 Step 8: Create Custom Alert (CPU > 80%)
In Grafana:

Dashboard → CPU Panel → Edit

Alert tab → Create Alert Rule

Condition:

avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) < 0.2


Set Evaluate every 1m

Add notification (Email / Slack)

✅ Alert triggers when CPU usage is high
