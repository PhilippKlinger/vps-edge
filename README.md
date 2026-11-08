# vps-edge

Shared edge proxy for all projects on the VPS: [Caddy](https://caddyserver.com/) terminates TLS and routes requests to the project stacks.

## Contents

- `compose.yaml`: Caddy (pinned by digest), the only service with published ports (80, 443, 443/udp), dropped capabilities
- `Caddyfile`: automatic HTTPS, admin API off, shared `security_headers` snippet, imports `sites/*.caddy`
- `sites/videoflix.caddy`: `/api/`, `/admin/`, `/static/`, API docs → `videoflix-web:8000` (body limit 55 MB), everything else → `videoflix-frontend:8080`
- `sites/status.caddy`: placeholder for `status.philipp-klinger.com` (503) until Uptime Kuma runs, so the name already has a valid certificate under HSTS `includeSubDomains`
- Docker network `edge`: project stacks join it as an external network and are reached by their network alias
- `host/journald-retention.conf`: deletes logs on the VPS after 14 days at the latest

Project stacks (e.g. [videoflix-deploy](https://github.com/PhilippKlinger/videoflix-deploy)) stay independent and replaceable; they never publish ports themselves. No access log is written, because activation and reset tokens are part of request paths.

## Usage on the server

```bash
cd /srv/edge
cp .env.example .env && chmod 600 .env   # set ACME_EMAIL
docker compose up -d
docker compose logs -f caddy             # certificate issuance
```

A new project gets its own file in `sites/`, then `docker compose restart caddy`. DNS for the domain must point to the VPS before the first start, otherwise the certificate request fails.

## Log retention

The privacy policy promises that server logs are deleted after 14 days at the latest. `json-file` can only rotate by size, so a quiet container could keep old entries for months. All containers therefore log to journald, which has a time limit. Caddy writes no access log and no log file, so there is nothing to rotate on its side; its error log can still contain client addresses, and that ends up in the journal too.

The setting applies to the whole journal of the host, so system logs are kept for 14 days as well. Set it up before the stacks switch to journald:

```bash
sudo install -d /etc/systemd/journald.conf.d
sudo cp /srv/edge/host/journald-retention.conf /etc/systemd/journald.conf.d/retention.conf
sudo systemctl restart systemd-journald
systemd-analyze cat-config systemd/journald.conf | grep -E 'MaxFileSec|MaxRetentionSec'
```

Then recreate the containers (`docker compose up -d` in each stack); this also removes their old `json-file` logs. Check:

```bash
docker inspect edge-caddy-1 --format '{{.HostConfig.LogConfig.Type}}'   # journald
journalctl --output=short-iso | head -n 2                               # oldest entry, 14 days at most
```

## Check

```bash
docker run --rm -e ACME_EMAIL=test@example.com -v "$PWD/Caddyfile:/etc/caddy/Caddyfile:ro" -v "$PWD/sites:/etc/caddy/sites:ro" caddy:2.11.7-alpine caddy validate --config /etc/caddy/Caddyfile
```
