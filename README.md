# MetaTrader 5 Quant Server (Docker)

This repository runs a full MetaTrader 5 stack with:

- MT5 + Wine + KasmVNC (`backend/mt5`)
- Django API (`backend/django`)
- PostgreSQL + Redis + Celery
- Traefik reverse proxy + automatic TLS
- Optional Grafana monitoring

## Project analysis (quick)

The orchestration is centered on `docker-compose.yml` and is production-oriented:

- `mt5` exposes MT5 desktop on VNC/web (port 3000) and MT5 API (port 5001) through Traefik.
- `django` uses `postgres` and `redis`, and background jobs run via `celery` + `celery-beat`.
- External access is expected through host-based routing (domains in `.env`) managed by Traefik.

## Debian upgrade

The MT5 container base image was upgraded from **Debian Bullseye** to **Debian Bookworm**:

- `ghcr.io/linuxserver/baseimage-kasmvnc:debianbookworm`
- WineHQ apt source changed to `bookworm`
- deprecated `apt-key` flow replaced with keyrings (`/etc/apt/keyrings`)

## Prerequisites

- Docker Engine 24+
- Docker Compose plugin (`docker compose`)
- Public DNS records for the domains configured in `.env`
- Open ports `80` and `443`

## Actual way to run the project

1. Clone and enter the repo:

   ```bash
   git clone https://github.com/sesto-dev/metatrader5-quant-server-python.git
   cd metatrader5-quant-server-python
   ```

2. Create env file:

   ```bash
   cp .env.example .env
   ```

3. Set required variables in `.env`:

   - `CUSTOM_USER`, `PASSWORD`
   - `VNC_DOMAIN`, `API_DOMAIN`, `DJANGO_DOMAIN`, `GRAFANA_DOMAIN`
   - `TRAEFIK_DOMAIN`, `TRAEFIK_USERNAME`, `ACME_EMAIL`

4. Generate Traefik password hash and export it for compose:

   ```bash
   export TRAEFIK_HASHED_PASSWORD="$(openssl passwd -apr1 'your-admin-password')"
   ```

5. Create the shared Traefik network (once per Docker host):

   ```bash
   docker network create traefik-public
   ```

6. Build and start all services:

   ```bash
   docker compose up -d --build
   ```

7. Verify containers are running:

   ```bash
   docker compose ps
   ```

8. Check MT5 logs during first boot:

   ```bash
   docker compose logs -f mt5
   ```

## Access

- MT5 web/VNC: `https://<VNC_DOMAIN>`
- MT5 API: `https://<API_DOMAIN>`
- Django: `https://<DJANGO_DOMAIN>`
- Traefik dashboard: `https://<TRAEFIK_DOMAIN>`
- Grafana: `https://<GRAFANA_DOMAIN>`

## Useful operations

Start/stop/restart:

```bash
docker compose up -d
docker compose down
docker compose restart mt5
```

Inspect logs:

```bash
docker compose logs -f
docker compose logs -f django mt5
```

Rebuild only MT5 image:

```bash
docker compose build --no-cache mt5
docker compose up -d mt5
```
