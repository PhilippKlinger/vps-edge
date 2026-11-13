# vps-edge

Shared edge proxy for all projects on the VPS: [Caddy](https://caddyserver.com/) terminates TLS and routes requests to the project stacks.

[![CI](https://github.com/PhilippKlinger/vps-edge/actions/workflows/ci.yml/badge.svg)](https://github.com/PhilippKlinger/vps-edge/actions/workflows/ci.yml)
[![Security Checks](https://github.com/PhilippKlinger/vps-edge/actions/workflows/security.yml/badge.svg)](https://github.com/PhilippKlinger/vps-edge/actions/workflows/security.yml)
[![CodeQL](https://github.com/PhilippKlinger/vps-edge/actions/workflows/codeql.yml/badge.svg)](https://github.com/PhilippKlinger/vps-edge/actions/workflows/codeql.yml)
[![OpenSSF Scorecard](https://api.scorecard.dev/projects/github.com/PhilippKlinger/vps-edge/badge)](https://scorecard.dev/viewer/?uri=github.com/PhilippKlinger/vps-edge)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Table of Contents

- [Overview](#overview)
- [Tech stack](#tech-stack)
- [CI and security](#ci-and-security)
- [Related repositories](#related-repositories)
- [License](#license)

## Overview

- Caddy terminates TLS for every project on the VPS, with automatic certificates
- Shared security headers for all sites: HSTS, `nosniff`, referrer policy, no `Server` header
- One file per project in `sites/`; project stacks join the network `edge` and never publish ports themselves
- Publishes only the public status page of [vps-monitoring](https://github.com/PhilippKlinger/vps-monitoring); the rest of Uptime Kuma stays unreachable
- Logs go to journald with a time limit of 14 days

Where the proxy sits in the platform: [platform diagram](https://github.com/PhilippKlinger/vps-monitoring#architecture) in vps-monitoring.

## Tech stack

- Caddy 2.11, pinned by digest
- Docker Compose, journald

## CI and security

- **Proxy test:** the CI starts the real configuration with certificates from a local CA and stand-in upstreams, and checks routing, the status page allowlist, the redirect to HTTPS and the security headers
- **Validation:** compose and Caddyfile validation, actionlint
- **Trivy scan** of the pinned Caddy image
- **Security checks:** secret scan (gitleaks), workflow lint (zizmor), CodeQL for the workflows

All checks are required before a merge. Validation and troubleshooting: [RUNBOOK.md](RUNBOOK.md).

## Related repositories

| Repository | Role |
|---|---|
| [videoflix-deploy](https://github.com/PhilippKlinger/videoflix-deploy) | Videoflix stack behind the proxy, network `edge` |
| [vps-monitoring](https://github.com/PhilippKlinger/vps-monitoring) | Uptime Kuma, its status page is published through this proxy (network `monitoring`) |
| [vps-incident-assistant](https://github.com/PhilippKlinger/vps-incident-assistant) | Uses RUNBOOK.md as runbook |
| [v-server-setup](https://github.com/PhilippKlinger/v-server-setup) | How the server is set up and hardened |

## License

[MIT](LICENSE)
