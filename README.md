# Flask Monitoring with Prometheus & Grafana

This repository is a hands-on monitoring setup for a Python Flask service using Prometheus, Grafana, and Prometheus Node Exporter running in Docker.

It is based on the original project [jmvaswani/metrics-demo](https://github.com/jmvaswani/metrics-demo), updated with fixes for Docker Compose v2 specification changes, corrected image registries, and proper internal container networking.

---

## What's in This Project?

When building web applications, you need visibility into both how your code is performing (HTTP request rates, latency, error status codes) and how the underlying host machine is holding up (CPU, memory, disk usage).

This project links together four core components:

1. **Flask API (`flask-app`)**: A simple Python Flask application instrumented with `prometheus-flask-exporter`. It automatically generates and exposes application metrics at `/metrics` on port 8080.
2. **Node Exporter (`node-exporter`)**: A lightweight daemon running on port 9100 that collects machine-level hardware and OS statistics.
3. **Prometheus (`prometheus`)**: A time-series database configured to scrape metrics from both the Flask app and Node Exporter every 3 seconds.
4. **Grafana (`grafana`)**: A dashboard tool listening on port 3000 to visualize the metrics stored in Prometheus.

---

## Screenshots

Here is what the setup looks like once everything is running:

### 1. Docker Containers

Docker containers running with no issues

![Docker container](success-screenshots/Screenshot%202026-10-18%20at%2018.39.36.png)

### 2. Prometheus Target Health

All scrape targets (`node-exporter`, `prometheus`, and `python-app`) reporting `UP`:

![Prometheus Targets](success-screenshots/Screenshot%202026-10-18%20at%2018.34.14.png)

### 3. Grafana Dashboard in Action

Visualizing CPU utilization, memory percentage, and live incoming HTTP requests by path and method:

![Grafana Dashboard](success-screenshots/Screenshot%202026-10-18%20at%2018.32.54.png)

---

## Quick Start

### Prerequisites

- Docker & Docker Compose v2 installed on your machine.

### 1. Spin up the containers

Clone the repo and start all services in the background:

```bash
docker compose up -d --build
```

### 2. Access the services

| Service           | Address                       | Notes                                          |
| :---------------- | :---------------------------- | :--------------------------------------------- |
| **Flask App**     | http://localhost:8080         | Sample routes: `/route1`, `/route2`            |
| **Flask Metrics** | http://localhost:8080/metrics | Prometheus metrics endpoint                    |
| **Node Exporter** | http://localhost:9100/metrics | Host & hardware metrics                        |
| **Prometheus**    | http://localhost:9090         | Check target health under **Status > Targets** |
| **Grafana**       | http://localhost:3000         | Default login: `admin` / `admin`               |

### 3. Shut down

```bash
docker compose down
```

---

## Setting Up Grafana

Once the containers are up, follow these steps to see your data:

### Step 1: Add Prometheus as a Data Source

1. Open [http://localhost:3000](http://localhost:3000) and sign in (`admin` / `admin`).
2. Go to **Connections** > **Data Sources** > click **Add data source**.
3. Choose **Prometheus**.
4. Set the **Prometheus server URL** to:
   ```
   http://prometheus:9090
   ```
   _(**Note**: Do not use `localhost:9090` here — Grafana runs in its own container, so `localhost` refers to Grafana itself. Docker's internal DNS lets containers talk using their service name: `http://prometheus:9090`)_.
5. Click **Save & test**. You should get a green confirmation.

### Step 2: Import the Dashboard

1. Go to **Dashboards** > **New** > **Import**.
2. Click **Upload dashboard JSON file** and select [`Metrics-Dashboard.json`](Metrics-Dashboard.json) from this repository.
3. Pick your Prometheus data source from the dropdown and hit **Import**.

---

## Simulating Traffic

To see the graphs update with live data, send a few test requests to the Flask server:

```bash
# Call route 1 (GET and POST)
curl http://localhost:8080/route1
curl -X POST http://localhost:8080/route1

# Call route 2 (GET and POST)
curl http://localhost:8080/route2
curl -X POST http://localhost:8080/route2

# Generate a small stream of requests
for i in {1..20}; do curl -s http://localhost:8080/route1 > /dev/null; done
```

Within a few seconds, the request count panels in your Grafana dashboard will reflect the incoming traffic.

---

## Key Takeaways & Gotchas

Working with Docker Compose and YAML configuration files often brings a few small stumbling blocks:

- **YAML indentation & comments**:
  - YAML uses strict 2-space indentation; never use tabs.
  - Comments must use `#`. C-style `//` comments are invalid in YAML and will break parsing.
- **Modern Docker Compose (v2)**:
  - The top-level `version:` tag is deprecated in Compose v2 and can simply be left out.
- **Port mapping indentation**:
  - Ensure port lists under `ports:` are placed on their own lines preceded by a dash (e.g., `- "9100:9100"`). Missing the `ports:` keyword causes Docker to mistakenly read the port number as part of the image name.
- **Container networking**:
  - Your browser reaches services using `http://localhost:<PORT>` because of host port mappings.
  - Containers talking to each other across the Docker network must use the service name defined in `docker-compose.yaml` (e.g. `http://prometheus:9090`, `http://flask-app:8080`).

---

## Acknowledgments

This repository builds upon the example provided by [Jatin Vaswani](https://github.com/jmvaswani) in [metrics-demo](https://github.com/jmvaswani/metrics-demo).

- **YAML indentation & comments**:
  - YAML uses strict 2-space indentation; never use tabs.
  - Comments must use `#`. C-style `//` comments are invalid in YAML and will break parsing.
- **Modern Docker Compose (v2)**:
  - The top-level `version:` tag is deprecated in Compose v2 and can simply be left out.
- **Port mapping indentation**:
  - Ensure port lists under `ports:` are placed on their own lines preceded by a dash (e.g., `- "9100:9100"`). Missing the `ports:` keyword causes Docker to mistakenly read the port number as part of the image name.
- **Container networking**:
  - Your browser reaches services using `http://localhost:<PORT>` because of host port mappings.
  - Containers talking to each other across the Docker network must use the service name defined in `docker-compose.yaml` (e.g. `http://prometheus:9090`, `http://flask-app:8080`).

---
