# vps-edge

Shared edge proxy for all projects on the VPS: [Caddy](https://caddyserver.com/) terminates TLS and routes requests to the project stacks.

## Planned contents

- Static `Caddyfile` with automatic HTTPS, admin API disabled
- Reusable security header snippet
- Docker network `edge`: project stacks join it, only Caddy publishes ports 80/443
- Redirects such as `backend-videoflix.` → `/api/docs/`

Project stacks (e.g. [videoflix-deploy](https://github.com/PhilippKlinger/videoflix-deploy)) stay independent and replaceable; they never publish ports themselves.

## Status

Repository set up, contents follow.
