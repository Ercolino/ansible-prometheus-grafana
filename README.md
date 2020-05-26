# Ansible Role: prometheus
Deploy Prometheus monitoring system using ansible.

## Requirements
- Ansible
- Tox

```
sudo apt install -y ansible
sudo apt install -y python-pip
pip install tox
```

## Steps
```
git clone https://github.com/cloudalchemy/ansible-prometheus
cd ansible-prometheus/
mkdir -p roles/cloudalchemy.prometheus
mv defaults/ handlers/ meta/ molecule/ tasks/ templates/ vars/ roles/cloudalchemy.prometheus
```

Create the playbook:
```
nano main.yaml
```

Then copy and paste the following code:
```
- hosts: all
  roles:
  - cloudalchemy.prometheus
  vars:
    prometheus_targets:
      node:
      - targets:
        - localhost:9100
        labels:
          env: demosite
```

Create the inventory:
```
nano inventory
```

Then copy and paste the following code:
```
localhost ansible_connection=local
```

Run the playbook:
```
ansible-playbook -i inventory main.yaml
```

Install Node Exporter for more metrics:
```
curl -LO  https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
tar xzf node_exporter-0.18.1.linux-amd64.tar.gz
cd node_exporter-0.18.1.linux-amd64/
mv node_exporter /usr/local/bin/
```

Create node_exporter service:
```
sudo adduser --no-create-home --disabled-login --shell /bin/false --gecos "Node Exporter User" node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
sudo vi /etc/systemd/system/node_exporter.service
```
Add the following code in node_exporter.service:
```
[Unit]
Description= Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Start the service and check the status:
```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

Modify the prometheus config file:
```
sudo vi /etc/prometheus/prometheus.yml
```

It should looks like the following(Change IP!):
```
global:
  evaluation_interval: 15s
  scrape_interval: 15s
  scrape_timeout: 10s

  external_labels:
    environment: ip-172-********.us-west-1.compute.internal

rule_files:
  - /etc/prometheus/rules/*.rules

scrape_configs:
  - job_name: prometheus
    metrics_path: /metrics
    static_configs:
    - targets:
      - ip-172-********.us-west-1.compute.internal:9090
  - file_sd_configs:
    - files:
      - /etc/prometheus/file_sd/node.yml
    job_name: node
  - job_name: 'node_exporter_metrics'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
```

Restart Prometheus and check status:
```
sudo systemctl restart prometheus
sudo systemctl status prometheus
```

Prometheus service listens on port 9090.

Install Grafana for graphing prometheus metrics:
```
cd ..
curl -LO https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana_5.1.4_amd64.deb
sudo apt-get install -y adduser libfontconfig
sudo dpkg -i grafana_5.1.4_amd64.deb
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

Grafana service listens on port 3000.