# GEMINI.md - Docker Home Lab Workspace

This workspace manages the Docker Compose stacks for a local home lab environment (IP: `192.168.29.116`). It uses Traefik as a reverse proxy to route traffic for `.lab` domains.

## 🏗 Architecture & Patterns

- **Reverse Proxy:** Traefik handles all incoming traffic.
- **HTTPS/TLS:** All services are served over HTTPS (port 443) by default with automatic redirection from HTTP (port 80).
- **Environment Management:** A root `.env` file manages common variables like `BASE_DOMAIN` and `CONFIG_ROOT`.
- **Networking:** All web-facing services MUST join the external Docker network named `proxy`.
- **DNS:** `*.lab` domains are routed via a Pi-hole instance at `192.168.29.43` to the Docker host.
- **Persistence:** Service configurations and data are stored in `${CONFIG_ROOT}/<service_name>`.
- **Deployment:** Each service is defined in its own YAML file within the `compose/` directory.

## 📋 Service Inventory

| Service | Compose File | Hostname | Description |
| :--- | :--- | :--- | :--- |
| **Traefik** | `traefik.yml` | `traefik.lab` | Core reverse proxy (HTTPS enabled) |
| **Docker Proxy** | `docker-proxy.yml` | `docker-proxy.lab` | Secure read-only socket proxy |
| **Grav** | `grav.yml` | `grav.lab` | Flat-file CMS |
| **Uptime Kuma** | `kuma.yml` | `kuma.lab` | Uptime & service monitoring |
| **Dashboard** | `nginx.yml` | `dashboard.lab` | Landing page (consolidated config) |
| **n8n** | `synopsis-station.yml` | `bigpi.lab:5678` | Workflow automation (non-root) |

## 🛠 Operational Conventions

### Managing Services
Always specify the compose file and the root `.env` file:
```bash
# Start a service
docker compose --env-file .env -f compose/kuma.yml up -d

# Stop a service
docker compose --env-file .env -f compose/kuma.yml down
```

### Adding New Services
When adding a new service to the stack:
1. **Environment:** Use `${BASE_DOMAIN}` and `${CONFIG_ROOT}` in the YAML.
2. **Network:** Ensure it uses the `proxy` network.
3. **Labels:** Add Traefik labels for HTTPS:
   ```yaml
   labels:
     - traefik.enable=true
     - traefik.http.routers.<name>.rule=Host(`<name>.${BASE_DOMAIN}`)
     - traefik.http.routers.<name>.entrypoints=websecure
     - traefik.http.routers.<name>.tls=true
     - traefik.http.services.<name>.loadbalancer.server.port=<internal_port>
   ```

### Critical Network Dependency
The `proxy` network is external and must exist before starting any stack:
```bash
docker network create proxy
```

## 📂 Directory Structure

- `compose/`: YAML definitions for Docker services.
- `configs/`: Persistent storage for services (not tracked in Git).
- `README.md`: Basic user-facing documentation.
- `GEMINI.md`: Architectural guidelines and LLM instructions (this file).
