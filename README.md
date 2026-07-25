# Homelab Monitoring Stack

A self-hosted lab environment for development, entertainment, and learning system administration. 


| Service | Purpose |
|---------|---------|
| **Nginx** | Reverse proxy and landing page. Single entry point on port 8080. |
| **Grafana** | Dashboard UI to visualize metrics, build panels, and set up alerts. |
| **Prometheus** | Time-series database that collects and stores metrics from all targets. |
| **Node Exporter** | Exposes Linux host metrics - CPU, RAM, disk, network, filesystems. |
| **cAdvisor** | Monitors resource usage of every running Docker container. |

## Prerequisites

- Linux host (Debian, Ubuntu, or any distro with Docker installed)
- Docker and Docker Compose v2 installed
- Ports 8080, 3000, 8081, 9090, and 9100 available

## Quick Start

```bash
git clone <your-repo-url>
cd docker-compose-sysadmin-lab
sudo docker compose up -d
```

Open your browser:

| URL | Service |
|-----|---------|
| `http://localhost:8080` | Homelab Dashboard |
| `http://localhost:8080/grafana/` | Grafana (via Nginx) |
| `http://localhost:8080/prometheus/` | Prometheus (via Nginx) |
| `http://localhost:3000` | Grafana (direct) |
| `http://localhost:9090` | Prometheus (direct) |
| `http://localhost:9100/metrics` | Node Exporter |
| `http://localhost:8081` | cAdvisor |

## Stopping

```bash
sudo docker compose down
```

To remove all stored data (metrics, dashboards, volumes):

```bash
sudo docker compose down -v
```

## Project Structure

```
docker-compose-sysadmin-lab/
├── compose.yaml                         # Service definitions
├── nginx/
│   ├── nginx.conf                       # Reverse proxy configuration
│   └── html/
│       └── index.html                   # Landing page
├── prometheus/
│   └── prometheus.yaml                  # Scrape targets configuration
└──grafana/
   └── provisioning/
       └── datasources/
           └── datasources.yaml         # (empty - add datasource manually)
```

## How It Works

### Reverse Proxy

Nginx listens on port 8080 and routes traffic based on the URL path:

- `/` serves the static landing page
- `/grafana/` proxies to Grafana on port 3000
- `/prometheus/` proxies to Prometheus on port 9090

All services communicate over a shared Docker network called `monitoring`.

### Prometheus Targets

Prometheus scrapes metrics every 15 seconds from:

| Target | Metrics Collected |
|--------|-------------------|
| `prometheus:9090` | Self-monitoring |
| `node-exporter:9100` | Host CPU, memory, disk, network |
| `cadvisor:8080` | Per-container resource usage |

### Startup Order

Nginx waits for both Grafana and Prometheus to pass their health checks before starting. All other services start independently.

## First-Time Grafana Setup

The Grafana datasource file is empty. After starting the stack:

1. Open Grafana at `http://localhost:8080/grafana/`
2. Log in with default credentials (`admin` / `admin`)
3. Go to **Connections > Data sources > Add data source**
4. Choose **Prometheus**
5. Set the URL to `http://prometheus:9090`
6. Click **Save & Test**

Once the datasource is added, you can build dashboards using Prometheus as the data source.

## Configuration Files

### nginx/nginx.conf

Defines the reverse proxy rules. Each `location` block maps a URL path to a backend service. Grafana's block includes WebSocket support for live features.

### prometheus/prometheus.yaml

Lists the scrape targets Prometheus collects from. Uses Docker service names (e.g., `node-exporter:9100`) for DNS resolution within the Docker network.

### grafana/provisioning/datasources/datasources.yaml

Currently empty. Used for auto-provisioning data sources on startup.

## Security Considerations

- All services run inside an isolated Docker network. Only Nginx is exposed to the host.
- Nginx config files and HTML are mounted as read-only (`:ro`) to prevent container-side changes.
- Prometheus and Grafana direct ports (9090, 3000) are exposed for development access. In a production setup, restrict these to localhost only or remove the port mappings entirely.
- Change the default Grafana admin password after first login.
- Consider adding HTTPS with Let's Encrypt or a self-signed certificate for external access.

## Future Scopes

### Short Term
- **Loki + Promtail** - Add log aggregation to complement Prometheus metrics. View container and system logs in Grafana.
- **Portainer** - Add a web UI for managing Docker containers, images, and volumes.
- **WireGuard or Tailscale** - Set up secure remote access to the homelab from outside your local network.
- **Traefik or Caddy** - Replace Nginx with an automatic HTTPS reverse proxy that handles TLS certificates.
- **Telegraf** - Collect additional system and application metrics with a flexible agent.
- **Alertmanager** - Configure Prometheus alert rules and route notifications to email, Telegram, or Discord.

### Long Term
- **Home Assistant** - Turn the homelab into a smart home hub with device automation.
- **Media Stack** - Add Jellyfin, Sonarr, Radarr, and Prowlarr for a self-hosted media server.
- **Game Servers** - Host Minecraft, Valheim, or other game servers alongside monitoring.
- **Forgejo** - Run a self-hosted Git platform with CI/CD pipelines.

