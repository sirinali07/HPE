## Observability 
In this lab, we'll set up Prometheus and Grafana on an independent EC2 instance. We'll also configure Prometheus to scrape metrics from a Ceph cluster.

### Prerequisites
* An EC2 instance (e.g., t3.xlarge) running Ubuntu.
* SSH access to the EC2 instance.
* Ceph cluster running with Prometheus module enabled.

### Task 1: SSH into the EC2 Instance
First, SSH into your EC2 instance.
```
ssh -i <your-key.pem> ubuntu@<ec2-public-ip>
```
Set Hostname
```
sudo hostnamectl hostname prometheus
```
### Task 2: Install and Configure Prometheus
Create Prometheus User and Directories
```
sudo useradd --no-create-home prometheus
```
```
sudo mkdir /etc/prometheus
```
```
sudo mkdir /var/lib/prometheus
```
Download and Install Prometheus
```
wget https://github.com/prometheus/prometheus/releases/download/v2.37.0/prometheus-2.37.0.linux-amd64.tar.gz
tar xvfz prometheus-2.37.0.linux-amd64.tar.gz prometheus-2.37.0.linux-amd64/
sudo cp prometheus-2.37.0.linux-amd64/prometheus /usr/local/bin
sudo cp prometheus-2.37.0.linux-amd64/promtool /usr/local/bin/
sudo cp -r prometheus-2.37.0.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus-2.37.0.linux-amd64/console_libraries /etc/prometheus
rm -rf prometheus-2.37.0.linux-amd64.tar.gz prometheus-2.37.0.linux-amd64
```
Edit the Prometheus configuration file.
```
sudo vi /etc/prometheus/prometheus.yml
```
Add the following configuration: Replace the `- targets IP` with the `Ceph Machines Private IP`
```yaml
global:
  scrape_interval: 15s
  external_labels:
    monitor: 'prometheus'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'ceph'
    static_configs:
      - targets: ['172.31.17.0:9283']
```
Create a systemd service file for Prometheus.
```
sudo vi /etc/systemd/system/prometheus.service
```
Add the following content:
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
Set Ownership and Permissions
```
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /var/lib/prometheus
```
Start and Enable Prometheus Service
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable prometheus
```
```
sudo service prometheus restart
```
```
sudo service prometheus status
```

### Task 3: Install and Configure Grafana
Install Grafana
```
sudo apt-get install -y apt-transport-https software-properties-common
```
```
sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
```
```
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
```
sudo apt-get update
```
```
sudo apt-get install grafana -y
```
Start and Enable Grafana Service
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable grafana-server.service
```
```
sudo systemctl start grafana-server
```
```
sudo systemctl status grafana-server
```

### Task 4: Configure Ceph to Export Metrics to Prometheus
On your Ceph cluster, enable the Prometheus module and configure it to export metrics:
```
sudo ceph mgr module enable prometheus
```
```
sudo ceph config set mgr mgr/prometheus/server_addr 172.31.17.0
```
```
sudo ceph config set mgr mgr/prometheus/server_port 9283
```
```
sudo ceph config dump | grep mgr
```
```
sudo ceph mgr services
```
```
sudo ceph status
```

### Task 5: Access Prometheus and Grafana
Access Prometheus at
```
http://<ec2-public-ip>:9090
```
Access Grafana at:
```
http://<ec2-public-ip>:3000
```
Login with default credentials (admin/admin) and set a new password.</br>
In Grafana, import the Ceph dashboard:

https://grafana.com/grafana/dashboards/9550-ceph-cluster/

Ensure that Prometheus is scraping the metrics from both Prometheus and Ceph by checking the targets in Prometheus:
```
http://<ec2-public-ip>:9090/targets
```
You should see both localhost:9090 (Prometheus) and 172.31.17.0:9283 (Ceph).

Conclusion
You have now successfully set up Prometheus and Grafana on an independent EC2 instance and configured it to scrape metrics from a Ceph cluster. You can visualize these metrics using Grafana dashboards.
