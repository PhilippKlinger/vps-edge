# vps-edge

Shared edge proxy for all projects on the VPS: [Caddy](https://caddyserver.com/) terminates TLS and routes requests to the project stacks.

## Contents

- `compose.yaml`: Caddy (pinned by digest), the only service with published ports (80, 443, 443/udp), dropped capabilities
- `Caddyfile`: automatic HTTPS, admin API off, shared `security_headers` snippet, imports `sites/*.caddy`
- `sites/videoflix.caddy`: `/api/`, `/admin/`, `/static/`, API docs → `videoflix-web:8000` (body limit 55 MB), everything else → `videoflix-frontend:8080`
- Docker network `edge`: project stacks join it as an external network and are reached by their network alias

Project stacks (e.g. [videoflix-deploy](https://github.com/PhilippKlinger/videoflix-deploy)) stay independent and replaceable; they never publish ports themselves. No access log is written, because activation and reset tokens are part of request paths.

## Usage on the server

```bash
cd /srv/edge
cp .env.example .env && chmod 600 .env   # set ACME_EMAIL
docker compose up -d
docker compose logs -f caddy             # certificate issuance
```

A new project gets its own file in `sites/`, then `docker compose restart caddy`. DNS for the domain must point to the VPS before the first start, otherwise the certificate request fails.

## Check

```bash
docker run --rm -e ACME_EMAIL=test@example.com -v "$PWD/Caddyfile:/etc/caddy/Caddyfile:ro" -v "$PWD/sites:/etc/caddy/sites:ro" caddy:2.11.7-alpine caddy validate --config /etc/caddy/Caddyfile
```
