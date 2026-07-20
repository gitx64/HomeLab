# SysAdmin Monitoring & Observability Lab

A professional, containerized laboratory environment for system administrators and DevOps engineers. This project provides a pre-configured stack for log analysis, infrastructure monitoring, and database management, all orchestrated behind an Nginx reverse proxy.

## 🚀 Overview

This lab demonstrates a modern observability stack that is fully compatible with both **Podman** and **Docker**. It features a custom Nginx dashboard that serves as a central gateway, handling complex subpath routing and proxy headers for a seamless experience.

### Included Services:
*   **Nginx Gateway**: Professional dashboard and reverse proxy (Port 8080).
*   **Kibana (8.14.0)**: Advanced data visualization and log analysis.
*   **Elasticsearch (8.14.0)**: High-performance search and analytics engine.
*   **Prometheus**: Time-series metrics collection and alerting.
*   **Node Exporter**: Hardware and OS metrics collector.
*   **PostgreSQL**: Reliable relational database for application data.

---

## 🏗️ Architecture

The environment uses a reverse proxy architecture to serve multiple applications on a single port (`8080`).

| Service | Access Path | Internal Port | Description |
| :--- | :--- | :--- | :--- |
| **Gateway** | `http://localhost:8080/` | 80 | Central Navigation Hub |
| **Kibana** | `http://localhost:8080/kibana/` | 5601 | Log Visualization (ELK) |
| **Prometheus** | `http://localhost:8080/prometheus/` | 9090 | Infrastructure Metrics |
| **Elasticsearch** | `http://localhost:9200` | 9200 | Backend Data Store |
| **Postgres** | `localhost:5432` | 5432 | Database Service |

---

## 🛠️ Setup & Installation

### Prerequisites
*   **Option A (Recommended):** [Podman](https://podman.io/) & [Podman-Compose](https://github.com/containers/podman-compose)
*   **Option B:** [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/)
*   **Resources:** Minimum 4GB RAM (Elasticsearch requirement).

### Quick Start
1.  **Clone the repository**:
    ```bash
    git clone <your-repo-url>
    cd docker-compose-sysadmin-lab
    ```

2.  **Start the environment**:
    ```bash
    # If using Podman
    podman-compose up -d

    # If using Docker
    docker-compose up -d
    ```

3.  **Access the Dashboard**:
    Open your browser and navigate to: [http://localhost:8080/](http://localhost:8080/)

---

## 🛡️ Why use Podman for this Lab?

While this project works perfectly on Docker, it is optimized for **Podman** for several technical and security reasons:

1.  **Rootless by Default**: Podman allows you to run this entire monitoring stack without `sudo` or root privileges. If a container is compromised, the attacker still only has the privileges of your non-root user.
2.  **Daemonless Architecture**: Unlike Docker, Podman doesn't rely on a central "big fat daemon" (the dockerd). Each container is a child process of the user's shell, making it more resilient and easier to audit.
3.  **Kubernetes Ready**: Podman can generate Kubernetes YAML manifests directly from your running pods (`podman generate kube`), making it a great stepping stone from Compose to K8s.
4.  **No Socket Risk**: Docker's `/var/run/docker.sock` is a frequent target for privilege escalation. Podman eliminates this attack vector entirely.

---

## 🔧 Technical Configuration Details

This lab includes several advanced configurations to ensure services work correctly behind a subpath:

*   **Nginx Trailing Slash Handling**: Configured with `absolute_redirect off;` to prevent the proxy from stripping the `8080` port during internal redirects.
*   **Kibana 8.x Proxying**: Uses `server.publicBaseUrl` and specific `X-Forwarded-*` headers to satisfy strict security/CSRF checks in the Elastic Stack.
*   **Prometheus Subpath**: Utilizes `--web.external-url` and `--web.route-prefix` to ensure UI assets (JS/CSS) load correctly through the proxy.

---

## ⚠️ Troubleshooting

*   **502 Bad Gateway**: Normal during the first 2-3 minutes of startup. Kibana takes time to initialize.
*   **Podman DNS**: If containers can't find each other, ensure you are using a Podman network (automatically handled by `podman-compose`).
*   **Virtual Memory**: Elasticsearch requires a high `vm.max_map_count`. If it crashes, run:
    ```bash
    sudo sysctl -w vm.max_map_count=262144
    ```

---

## 📝 License
This project is for educational purposes. Feel free to use and modify it for your own labs.
