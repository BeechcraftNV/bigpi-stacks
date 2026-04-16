# Home Lab Docker Stacks

This repository contains the Docker Compose configurations and management patterns for my home lab server (`192.168.29.116`).

## 🏗 Architecture Overview

- **Reverse Proxy:** [Traefik v3.1](https://doc.traefik.io/traefik/) serves as the edge router.
- **Entrypoints:**
    - `web` (Port 80): Automatically redirects to `websecure`.
    - `websecure` (Port 443): Handles all traffic with TLS enabled.
- **Networking:** All web-facing services join an external Docker network named `proxy`.
- **DNS:** Local `.lab` domains are resolved via Pi-hole (`192.168.29.43`).
- **Persistence:** Application data and configurations are stored in `/opt/docker/configs/`.

## 📁 Directory Structure

```text
/opt/docker/
├── compose/           # Docker Compose YAML definitions
├── configs/           # Persistent service data (not tracked in Git)
├── .env               # Centralized environment variables
├── GEMINI.md          # Architectural standards for LLM agents
└── README.md          # This file
```

## 📋 Service Inventory

| Service | Hostname | Internal Port | Description |
| :--- | :--- | :--- | :--- |
| **Traefik** | `traefik.lab` | 8080 (API) | Reverse proxy & dashboard |
| **Grav** | `grav.lab` | 80 | Flat-file CMS |
| **Uptime Kuma** | `kuma.lab` | 3001 | Service monitoring & status pages |
| **Dashboard** | `dashboard.lab` | 80 | Nginx-based internal landing page |
| **n8n** | `bigpi.lab` | 5678 | Workflow automation (Youtube Synopses) |

## 🚀 Getting Started

### 1. Prerequisites
Ensure the external `proxy` network exists before starting any services:
```bash
docker network create proxy
```

### 2. Environment Configuration
Create a `.env` file in the root directory based on your environment:
```env
BASE_DOMAIN=lab
CONFIG_ROOT=/opt/docker/configs
TZ=America/Los_Angeles
PUID=1000
PGID=1000
```

### 3. Managing Services
All commands should be run from the root of this repository using the `--env-file` flag:

```bash
# Start a service (e.g., Uptime Kuma)
docker compose --env-file .env -f compose/kuma.yml up -d

# Stop a service
docker compose --env-file .env -f compose/kuma.yml down

# View logs
docker compose --env-file .env -f compose/kuma.yml logs -f
```

## 🛠 Adding New Services

To add a new service to the stack, ensure it follows these conventions:

1.  **Compose File:** Create a new YAML in `compose/`.
2.  **Network:** Join the external `proxy` network.
3.  **Labels:** Add Traefik labels for automatic routing and TLS:
    ```yaml
    labels:
      - traefik.enable=true
      - traefik.http.routers.<name>.rule=Host(`<name>.${BASE_DOMAIN}`)
      - traefik.http.routers.<name>.entrypoints=websecure
      - traefik.http.routers.<name>.tls=true
      - traefik.http.services.<name>.loadbalancer.server.port=<port>
    ```

## 📦 Git & Maintenance

This repository is designed to track **configurations only**.

- **`.gitignore`:** Ensure `configs/` and `.env` are excluded from version control to protect sensitive data and prevent bloat.
- **Backups:** Regularly back up the `/opt/docker/configs/` directory separately, as it contains all persistent database and application state.
