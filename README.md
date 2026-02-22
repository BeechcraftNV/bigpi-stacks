# docker

Docker Compose definitions for the home lab server at 192.168.29.116.

## Structure

```
compose/   # Docker Compose service definitions
configs/   # Runtime data and volumes (not tracked in git)
```

## Services

| Service | Hostname | Description |
|---|---|---|
| Traefik | traefik.lab/dashboard/ | Reverse proxy — routes *.lab traffic |
| Uptime Kuma | kuma.lab | Uptime monitoring |

## Network

- DNS: Pi-hole at 192.168.29.43 routes `*.lab` → 192.168.29.116:80
- All services share the external Docker network `proxy`

## Usage

Start a service:
```bash
docker compose -f compose/<file>.yml up -d
```

The `proxy` network must exist before starting any service:
```bash
docker network create proxy
```
