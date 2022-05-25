# Prometheus

[Prometheus](https://prometheus.io/) is a time series database for monitoring systems, this will serve as the backend of the monitoring service. This service has an official image on [Docker Hub](https://hub.docker.com/r/prom/prometheus) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/monitoring/prometheus
```

Inside this folder, create a `data` folder:

```bash
mkdir data
```

And also, create a `prometheus.yml` file:

```bash
nano prometheus.yml
```

Which should have the following:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    scrape_interval: 15s
    static_configs:
    - targets: ["localhost:9090"]

  - job_name: "node"
    static_configs:
    - targets: ["host.docker.internal:9100"]
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  exporter:
    image: prom/node-exporter:latest
    restart: unless-stopped
    user: '1000:1000'
    network_mode: host
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'

  prometheus:
    image: prom/prometheus:latest
    restart: unless-stopped
    user: '1000:1000'
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    ports:
      - 9090:9090
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
```

!!! note
    The `exporter` service needs to have `network_mode: host` in order to have access to the main network interface to actually have valid network usage data.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 9090/tcp
sudo ufw allow 9100/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
