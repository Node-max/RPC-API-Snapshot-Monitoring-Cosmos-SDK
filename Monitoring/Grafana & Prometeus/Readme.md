# Grafana & Prometeus
## Install monitoring stack
- **install the monitoring using the auto installer from kjnode**
```python
wget -O install_monitoring.sh https://raw.githubusercontent.com/kj89/cosmos_node_monitoring/master/install_monitoring.sh && chmod +x install_monitoring.sh && ./install_monitoring.sh
```
## Setting Config Node
- **Changing Node Configs**
- 
![image](https://user-images.githubusercontent.com/61777095/223358599-366a3f43-d027-48ab-baf3-f81671c7aed2.png)

Follow the instructions below
**step into**
```python
nano .nolus/config/config.toml
```
**change prometheus = false to true**
**and save both prometheus_listen_addr = ":43660"**
**then restart the nolus node using the following command**
```python
sudo systemctl restart nolusd
```
## Changing the Prometheus Config
- **Follow the instructions below**
**press on the keyboard cd**
**then enter the following command**
```python
nano cosmos_node_monitoring/prometheus/prometheus.yml
```
![image](https://user-images.githubusercontent.com/61777095/223359656-a2d1686e-740b-483d-83a7-008d5c5ef016.png)

**delete all contents by pressing on the keyboard SHIFT+DOWN ARROW for the block code then pressing on the keyboard CTRL+K**

![image](https://user-images.githubusercontent.com/61777095/223359769-84c108d1-4139-48ce-8b53-69461c01d908.png)

**if it has been deleted, change it with the following config**
```python
global:
  scrape_interval:     15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['139.59.242.182:9090']
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['139.59.242.182:9100']
  - job_name: "nolus-testnet"
    metrics_path: '/metrics'
    static_configs:
      - targets: ["139.59.242.182:43660"] # custom with your own!
```
![image](https://user-images.githubusercontent.com/61777095/223359937-51c49937-f64a-4d33-a1c7-b3b75677e397.png)

**if you have pressed on the keyboard CTRL + X then press Y then ENTER**
## Enable Firewalls
- **If you don't have a firewall installed, install the ufw firewall with the command:**
```python
sudo apt-get install ufw
```
**Create firewall rules to allow incoming traffic on ports 9100, 43660, and 9090 with the command:**

```python
sudo ufw allow 9100/tcp
sudo ufw allow 43660/tcp
sudo ufw allow 9090/tcp
```

**The above command will create a new firewall rule that allows incoming traffic on the specified port.**
**Activate ufw with the command:**
```python
sudo ufw enable
```
**The above command will enable ufw and start the firewall.**
## Change the Config docker-compose.yml 
**At this stage we will change the docker-compose.yml config to disable the login feature**
- **Make sure you are outside the folder by typing cd then enter the following command**
```python
nano cosmos_node_monitoring/docker-compose.yml
```
**then delete using the keyboard SHIFT+DIRECTION ARROW DOWN for the block code then press on the keyboard CTRL+K**
```python
version: "3.5"

volumes:
  prometheus_data: {}
  grafana_data: {}
  alerta_db: {}

networks:
  cosmos-monitoring:

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
       - type: volume
         source: prometheus_data
         target: /prometheus
       - type: bind
         source: ./prometheus
         target: /etc/prometheus
         read_only: true
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
    ports:
      - 9090:9090
    networks:
      - cosmos-monitoring
    restart: always

  # default login credentials: admin/admin
  grafana:
    image: grafana/grafana:latest
    env_file: ./grafana/grafana.conf
    container_name: grafana
    volumes:
      - type: volume
        source: grafana_data
        target: /var/lib/grafana
      - ./grafana/datasource.yml:/etc/grafana/provisioning/datasources/datasource.yml
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
    ports:
      - 9999:3000
    networks:
      - cosmos-monitoring
    restart: always

    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
```
**if you have pressed on the keyboard CTRL + X then press Y then ENTER**
## Run Docker

![image](https://user-images.githubusercontent.com/61777095/223361327-5e987167-a9b0-413f-aff8-14738e398610.png)

**Run docker using the following command**
```python
cd $HOME/cosmos_node_monitoring
sudo docker compose up -d
```
## Make sure your prometheus is green

<img width="514" alt="image" src="https://user-images.githubusercontent.com/61777095/223361612-0a002beb-38ce-479c-a8dc-a21b896cd7ab.png">

**To enter Prometheus you can change the following domains, make sure everything is green!**
http://ipserver:9090/targets?search=
## Create Dashboard Monitor
**first you have to login to your domain**
http://ipserver:9999/login

<img width="354" alt="image" src="https://user-images.githubusercontent.com/61777095/223362351-ddb67740-4dbb-418d-8676-8f535c0d0946.png">

**Enter username and password using admin**

<img width="381" alt="image" src="https://user-images.githubusercontent.com/61777095/223362486-6d0ff04b-01cc-445b-8cf4-a4b8ff2c26be.png">

**Create data sources**
**please read this carefully**

<img width="477" alt="image" src="https://user-images.githubusercontent.com/61777095/223362696-48d42175-182b-4bde-b359-cd5b09f8aa3f.png">

<img width="474" alt="image" src="https://user-images.githubusercontent.com/61777095/223362745-14e414cd-e0de-4aa1-bca7-41b8667433a5.png">

<img width="474" alt="image" src="https://user-images.githubusercontent.com/61777095/223362787-9c3f0969-af6e-4e56-b8b6-684acf339d90.png">

- **In the name section, please fill in what you want the name or whatever**
- **while url use http://ipserver:9090/**
- **then scroll down press save and test**
## Create a dashboard view

<img width="421" alt="image" src="https://user-images.githubusercontent.com/61777095/223363262-ab3891a2-12f6-4106-8501-7e48d7404d6c.png">

**hover over the box image**
**then select import**

<img width="428" alt="image" src="https://user-images.githubusercontent.com/61777095/223363449-67a769c7-e81f-4f3f-8527-e25292fb73d2.png">

**in the Import section via the json panel fill in the following code**
```python
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "target": {
          "limit": 100,
          "matchAny": false,
          "tags": [],
          "type": "dashboard"
        },
        "type": "dashboard"
      }
    ]
  },
  "description": "Dashboard for tendermint, go prometheus metrics exported via the cosmos-sdk.",
  "editable": true,
  "fiscalYearStartMonth": 0,
  "gnetId": 17100,
  "graphTooltip": 2,
  "id": 4,
  "links": [
    {
      "asDropdown": true,
      "icon": "external link",
      "includeVars": true,
      "keepTime": true,
      "tags": [],
      "targetBlank": true,
      "title": "Switch",
      "tooltip": "",
      "type": "dashboards",
      "url": ""
    }
  ],
  "liveNow": true,
  "panels": [
    {
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 0
      },
      "id": 4,
      "panels": [],
      "title": "Node Health",
      "type": "row"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "/status sync_info number",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 6,
        "x": 0,
        "y": 1
      },
      "id": 10,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.4.3",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "tendermint_consensus_latest_block_height{job=\"nolus-testnet\"}",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Block Height",
      "transparent": true,
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-GrYlRd"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 5,
        "w": 6,
        "x": 6,
        "y": 1
      },
      "id": 40,
      "options": {
        "displayMode": "lcd",
        "minVizHeight": 10,
        "minVizWidth": 0,
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showUnfilled": true
      },
      "pluginVersion": "9.4.3",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "process_open_fds{job=\"nolus-testnet\"}",
          "hide": false,
          "legendFormat": "open_fds",
          "range": true,
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "process_max_fds{instance=\"$instance\"}",
          "instant": false,
          "legendFormat": "Process Max fds",
          "range": true,
          "refId": "B"
        }
      ],
      "title": "Number of open file descriptors",
      "transparent": true,
      "type": "bargauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Total open file descriptors",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 10,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 2,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": true,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "short"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 12,
        "y": 1
      },
      "id": 3,
      "links": [],
      "options": {
        "legend": {
          "calcs": [
            "mean",
            "lastNotNull",
            "max"
          ],
          "displayMode": "table",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "multi",
          "sort": "none"
        }
      },
      "pluginVersion": "9.0.3-0dc0087e",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "process_open_fds{job=\"nolus-testnet\"}",
          "format": "time_series",
          "intervalFactor": 2,
          "legendFormat": "num open fds",
          "metric": "process_open_fds",
          "range": true,
          "refId": "A",
          "step": 4
        }
      ],
      "title": "Open File Descriptors",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {},
      "description": "Running status of instance",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [
            {
              "options": {
                "0": {
                  "color": "red",
                  "index": 0,
                  "text": "Down"
                },
                "1": {
                  "color": "green",
                  "index": 1,
                  "text": "Up"
                }
              },
              "type": "value"
            }
          ],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 6,
        "w": 5,
        "x": 0,
        "y": 5
      },
      "id": 17,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.4.3",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "${DS_PROMETHEUS}"
          },
          "editorMode": "code",
          "expr": "up{job=\"nolus-testnet\"}",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Up",
      "transparent": true,
      "type": "stat"
    },
    {
      "datasource": {},
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-GrYlRd"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "decbytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 6,
        "w": 6,
        "x": 6,
        "y": 6
      },
      "id": 57,
      "options": {
        "displayMode": "lcd",
        "minVizHeight": 10,
        "minVizWidth": 0,
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showUnfilled": true
      },
      "pluginVersion": "9.4.3",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "${DS_PROMETHEUS}"
          },
          "editorMode": "code",
          "expr": "process_virtual_memory_bytes{job=\"nolus-testnet\"}",
          "hide": false,
          "legendFormat": "virtual_memory",
          "range": true,
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "${DS_PROMETHEUS}"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "process_virtual_memory_max_bytes{instance=\"$instance\"}",
          "instant": false,
          "legendFormat": "virtual_memory_max",
          "range": true,
          "refId": "B"
        }
      ],
      "title": "Virtual memory size",
      "transparent": true,
      "type": "bargauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-GrYlRd"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "decbytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 3,
        "w": 12,
        "x": 12,
        "y": 8
      },
      "id": 59,
      "options": {
        "displayMode": "lcd",
        "minVizHeight": 10,
        "minVizWidth": 0,
        "orientation": "horizontal",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "showUnfilled": true
      },
      "pluginVersion": "9.4.3",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "go_memstats_alloc_bytes{job=\"nolus-testnet\"}",
          "instant": false,
          "legendFormat": "alloc_bytes",
          "range": true,
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "go_memstats_alloc_bytes_total{instance=\"$instance\"}",
          "hide": false,
          "legendFormat": "alloc_bytes_total",
          "range": true,
          "refId": "B"
        }
      ],
      "title": "Number of bytes allocated",
      "transparent": true,
      "type": "bargauge"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 25,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "normal"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "decbytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 11,
        "w": 12,
        "x": 12,
        "y": 11
      },
      "id": 48,
      "options": {
        "legend": {
          "calcs": [
            "mean",
            "last",
            "max"
          ],
          "displayMode": "table",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "9.1.6",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "go_memstats_heap_alloc_bytes{job=\"nolus-testnet\"}",
          "instant": false,
          "legendFormat": "go_memstats_heap_alloc(bytes)",
          "range": true,
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "go_memstats_heap_inuse_bytes{instance=\"$instance\"}",
          "hide": false,
          "legendFormat": "go_memstats_heap_inuse",
          "range": true,
          "refId": "B"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "go_memstats_heap_idle_bytes{instance=\"$instance\"}",
          "hide": false,
          "legendFormat": "go_memstats_heap_idle",
          "range": true,
          "refId": "C"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "go_memstats_heap_sys_bytes{instance=\"$instance\"}",
          "hide": false,
          "legendFormat": "go_memstats_heap_sys_bytes",
          "range": true,
          "refId": "D"
        }
      ],
      "title": "Heap Stats",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Resident and Virtual memory size in bytes",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 10,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 2,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": true,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "bytes"
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "resident"
            },
            "properties": [
              {
                "id": "unit",
                "value": "short"
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 12
      },
      "id": 1,
      "links": [],
      "options": {
        "legend": {
          "calcs": [
            "mean",
            "lastNotNull",
            "max"
          ],
          "displayMode": "table",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "multi",
          "sort": "none"
        }
      },
      "pluginVersion": "9.0.3-0dc0087e",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "process_resident_memory_bytes{job=\"nolus-testnet\"}",
          "format": "time_series",
          "intervalFactor": 2,
          "legendFormat": "{{pod}} - resident",
          "metric": "process_resident_memory_bytes",
          "range": true,
          "refId": "A",
          "step": 4
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "process_virtual_memory_bytes{namespace=~\"^($namespace)$\", instance=\"$instance\"}",
          "format": "time_series",
          "intervalFactor": 2,
          "legendFormat": "{{pod}} - virtual",
          "metric": "process_virtual_memory_bytes",
          "range": true,
          "refId": "B",
          "step": 4
        }
      ],
      "title": "Memory Utilization",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Number of goroutines that currently exist",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 10,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 2,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": true,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "short"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 0,
        "y": 20
      },
      "id": 7,
      "links": [],
      "options": {
        "legend": {
          "calcs": [
            "mean",
            "lastNotNull",
            "max"
          ],
          "displayMode": "table",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "multi",
          "sort": "none"
        }
      },
      "pluginVersion": "9.0.3-0dc0087e",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "go_goroutines{job=\"nolus-testnet\"}",
          "format": "time_series",
          "intervalFactor": 2,
          "legendFormat": "num goroutines",
          "metric": "go_goroutines",
          "range": true,
          "refId": "A",
          "step": 4
        }
      ],
      "title": "Total Goroutines",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Total transactions",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-GrYlRd"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 20,
            "gradientMode": "scheme",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "smooth",
            "lineWidth": 3,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 12,
        "w": 12,
        "x": 12,
        "y": 22
      },
      "id": 26,
      "options": {
        "legend": {
          "calcs": [
            "last"
          ],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "9.1.6",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "tendermint_consensus_total_txs{job=\"nolus-testnet\"}",
          "legendFormat": "consensus total txs",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Total Transactions",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Number of bytes used for other system allocations",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-GrYlRd"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 20,
            "gradientMode": "scheme",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "smooth",
            "lineWidth": 3,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "decbytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 7,
        "w": 12,
        "x": 0,
        "y": 27
      },
      "id": 54,
      "options": {
        "legend": {
          "calcs": [
            "mean",
            "last",
            "max"
          ],
          "displayMode": "table",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "go_memstats_other_sys_bytes{job=\"nolus-testnet\"}",
          "instant": false,
          "legendFormat": "memstats_other_sys",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Other Run Time Allocations Memory Usage",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "collapsed": false,
      "gridPos": {
        "h": 1,
        "w": 24,
        "x": 0,
        "y": 34
      },
      "id": 2,
      "panels": [],
      "title": "Tendermint",
      "type": "row"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "dark-red",
                "value": 10
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 6,
        "x": 6,
        "y": 35
      },
      "id": 9,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.4.3",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "tendermint_consensus_height{job=\"nolus-testnet\"}- tendermint_consensus_latest_block_height{job=\"nolus-testnet\"} - 1",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Missed Blocks",
      "transparent": true,
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Number of validators",
      "fieldConfig": {
        "defaults": {
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "red"
              },
              {
                "color": "green",
                "value": 10
              }
            ]
          },
          "unit": "short"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 4,
        "x": 12,
        "y": 35
      },
      "id": 25,
      "options": {
        "colorMode": "value",
        "graphMode": "area",
        "justifyMode": "auto",
        "orientation": "auto",
        "reduceOptions": {
          "calcs": [
            "lastNotNull"
          ],
          "fields": "",
          "values": false
        },
        "textMode": "auto"
      },
      "pluginVersion": "9.4.3",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "tendermint_consensus_validators{job=\"nolus-testnet\"}",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Consensus Validators",
      "transparent": true,
      "type": "stat"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Number of peers node is connected to",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [
            {
              "options": {
                "0": {
                  "color": "red",
                  "index": 0,
                  "text": "Down"
                },
                "1": {
                  "color": "green",
                  "index": 1,
                  "text": "Up"
                }
              },
              "type": "value"
            }
          ],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 4,
        "w": 8,
        "x": 16,
        "y": 35
      },
      "id": 18,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "9.0.3-0dc0087e",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "tendermint_p2p_peers{job=\"nolus-testnet\"}",
          "legendFormat": "num peers",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Total Peers",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Number of transactions",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": [
          {
            "__systemRef": "hideSeriesFrom",
            "matcher": {
              "id": "byNames",
              "options": {
                "mode": "exclude",
                "names": [
                  "num txs"
                ],
                "prefix": "All except:",
                "readOnly": true
              }
            },
            "properties": [
              {
                "id": "custom.hideFrom",
                "value": {
                  "legend": false,
                  "tooltip": false,
                  "viz": true
                }
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 39
      },
      "id": 16,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "tendermint_consensus_num_txs{job=\"nolus-testnet\"}",
          "legendFormat": "num txs",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Number of Transactions",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Number of bytes per channel sent to a given peer",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "Bandwidth",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 4,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineStyle": {
              "fill": "solid"
            },
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "decbytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 39
      },
      "id": 13,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "right",
          "showLegend": false
        },
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "sum by (peer_id) (rate(tendermint_p2p_peer_send_bytes_total{job=\"nolus-testnet\"}[15m]))",
          "legendFormat": "peer {{peer_id}}",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Peer Network Output",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Number of uncommitted transactions",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 2,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": [
          {
            "matcher": {
              "id": "byName",
              "options": "failed tx rate"
            },
            "properties": [
              {
                "id": "color",
                "value": {
                  "fixedColor": "dark-red",
                  "mode": "fixed"
                }
              }
            ]
          }
        ]
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 47
      },
      "id": 15,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "tendermint_mempool_size{job=\"nolus-testnet\"}",
          "legendFormat": "num of uncommitted txs",
          "range": true,
          "refId": "A"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "rate(tendermint_mempool_failed_txs[5m])",
          "hide": false,
          "legendFormat": "failed tx rate",
          "range": true,
          "refId": "B"
        },
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "",
          "hide": false,
          "legendFormat": "__auto",
          "range": true,
          "refId": "C"
        }
      ],
      "title": "Uncommitted Txs",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Transaction sizes in bytes",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 0,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 47
      },
      "id": 23,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "exemplar": false,
          "expr": "tendermint_mempool_tx_size_bytes_count{job=\"nolus-testnet\"}",
          "format": "time_series",
          "instant": false,
          "interval": "",
          "legendFormat": "",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Mempool Tx Size",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Number of bytes per channel received from a given peer",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "Bandwidth",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 3,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineStyle": {
              "fill": "solid"
            },
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "decbytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 55
      },
      "id": 6,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": false
        },
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "sum by (peer_id) (rate(tendermint_p2p_peer_receive_bytes_total{job=\"nolus-testnet\"}[10m]))",
          "legendFormat": "peer {{peer_id}}",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Peer Network Input",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Sum of blockparts transmitted by peer in 5m timeframe",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "Bandwidth",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 3,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineStyle": {
              "fill": "solid"
            },
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "decbytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 55
      },
      "id": 34,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": false
        },
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "sum by (peer_id) (rate(tendermint_consensus_block_parts{job=\"nolus-testnet\"}[5m]))",
          "legendFormat": "peer {{peer_id}}",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Consensus Block Parts",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Number of pending bytes to be sent to a given peer",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "Bandwidth",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 3,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineStyle": {
              "fill": "solid"
            },
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "decbytes"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 63
      },
      "id": 21,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": false
        },
        "tooltip": {
          "mode": "multi",
          "sort": "desc"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "builder",
          "expr": "{address=\"02:42:57:73:fd:b1\"}",
          "legendFormat": "__auto",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Peer Network Output - Pending",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Number of transactions rechecked in the mempool",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "continuous-GrYlRd"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 20,
            "gradientMode": "scheme",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "smooth",
            "lineWidth": 3,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          }
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 63
      },
      "id": 27,
      "options": {
        "legend": {
          "calcs": [
            "last"
          ],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "9.1.6",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "tendermint_mempool_recheck_times{job=\"nolus-testnet\"}",
          "legendFormat": "Mempool Recheck Times",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Mempool Recheck Times",
      "transparent": true,
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "PBFA97CFB590B2093"
      },
      "description": "Difference in seconds between the proposal timestamp and the timestamp of the latest prevote that achieved a quorum in the prevote step",
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 25,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "auto",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green"
              },
              {
                "color": "red",
                "value": 80
              }
            ]
          },
          "unit": "s"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 71
      },
      "id": 31,
      "options": {
        "legend": {
          "calcs": [
            "last"
          ],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "pluginVersion": "9.1.6",
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "PBFA97CFB590B2093"
          },
          "editorMode": "code",
          "expr": "tendermint_consensus_quorum_prevote_message_delay{job=\"nolus-testnet\"}",
          "legendFormat": "consensus_quorum_prevote_message_delay",
          "range": true,
          "refId": "A"
        }
      ],
      "title": "Prevote Message Delay",
      "transparent": true,
      "type": "timeseries"
    }
  ],
  "refresh": "5s",
  "revision": 1,
  "schemaVersion": 38,
  "style": "dark",
  "tags": [
    "cosmos",
    "tendermint",
    "validator",
    "okp4"
  ],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-30m",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "",
  "title": "SPT-NOLUS",
  "uid": "SPT-NODE-NOLUS",
  "version": 2,
  "weekStart": ""
}
```

<img width="429" alt="image" src="https://user-images.githubusercontent.com/61777095/223363629-e5847a4a-db6b-4688-be8d-ed1f827c9718.png">

**in this section you can change the name and uid to your heart's content**
**then hit import**

<img width="960" alt="image" src="https://user-images.githubusercontent.com/61777095/223363907-ff54451f-ca4a-401f-855c-5fc2ba7733d4.png">


