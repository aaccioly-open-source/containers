<!--
SPDX-FileCopyrightText: 2026 Anthony Accioly <anthony@accioly.dev>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Authentik

This directory contains a curated Docker Compose deployment for [Authentik](https://goauthentik.io/) that is aligned with the official [Docker Compose installation guide](https://docs.goauthentik.io/install-config/install/docker-compose/) while adapting the local paths to this repository.

## About

This stack includes:

- **PostgreSQL 16** — bundled database for Authentik
- **authentik server** — the web UI, API, and login flows
- **authentik worker** — background jobs and outpost management

The compose file is pre-wired for the repository’s `nginx-proxy` stack through `VIRTUAL_HOST`, `LETSENCRYPT_HOST`, and `VIRTUAL_PORT` environment variables.

## Layout

- `compose.yml` — the service definition
- `env.example` — safe template for local secrets and runtime settings
- `.env` — local secrets and runtime settings copied from `env.example`
- `data/` — Authentik application data mounted into `/data`
- `config/custom-templates/` — optional custom templates mounted into `/templates`
- `config/certs/` — certificate material mounted into the worker container at `/certs`

## Prerequisites

1. Docker Compose or Podman Compose
2. The external `nginx-proxy` network, if you plan to expose Authentik through the reverse proxy
3. A valid SMTP provider if you want outbound email, password resets, or notifications
4. Access to a container runtime socket for the worker container

If you are using Podman, make sure the socket is enabled before starting the stack:

```bash
systemctl --user enable --now podman.socket
```

For rootful Podman, use `sudo systemctl enable --now podman.socket` instead.

## Quick start

1. Copy the example environment file:

   ```bash
   cp env.example .env
   ```

2. Generate strong secrets and paste them into `.env`:

   ```bash
   openssl rand -base64 36
   openssl rand -base64 60
   ```

   Use the first value for `PG_PASS` and the second for `AUTHENTIK_SECRET_KEY`.

3. Edit `.env` and set at least:

   - `PG_PASS`
   - `AUTHENTIK_SECRET_KEY`
   - `VIRTUAL_HOST` and `LETSENCRYPT_HOST` if you are using `nginx-proxy`
   - `AUTHENTIK_EMAIL__*` if you want email notifications
   - `CONTAINER_SOCKET` if you are not using the default Docker socket path

4. Start the stack:

   ```bash
   podman compose up -d
   ```

   or:

   ```bash
   docker compose up -d
   ```

5. Watch the logs:

   ```bash
   podman compose logs -f
   ```

## First-time login

After the containers are healthy, open Authentik at:

- `http://localhost:9001` for direct access in this repository’s default port mapping
- `https://localhost:9443` if you want to test the published TLS port directly
- `https://authentik.example.com` or your real hostname if you are fronting the service with `nginx-proxy`

Authentik will prompt you to complete the initial setup for the built-in `akadmin` user. Set the initial password, then continue with the first-login wizard.

## Reverse proxy notes

This repository already connects the Authentik server container to the external `nginx-proxy` network. When using the reverse proxy:

- `VIRTUAL_HOST` should match the public hostname, for example `authentik.example.com`
- `LETSENCRYPT_HOST` should usually match `VIRTUAL_HOST`
- `VIRTUAL_PORT` should stay on `9000`, because that is the container port that `nginx-proxy` should reach internally

If you are not using the reverse proxy, you can leave those variables unset or ignore them.

## Email configuration

Global email settings are optional but strongly recommended. Authentik uses them for:

- password resets
- administrator alerts
- configuration warnings
- release notifications
- email stages

Set the SMTP values in `.env` to match your provider. If your SMTP relay does not require authentication, you can leave `AUTHENTIK_EMAIL__USERNAME` and `AUTHENTIK_EMAIL__PASSWORD` empty.

## Updating

To update Authentik, use `AUTHENTIK_TAG` in `.env` as the starting point, but do not assume a tag bump alone will be enough. 
Before upgrading, compare the upstream [`compose.yml`](https://docs.goauthentik.io/compose.yml) for the target release against the version you are running now, 
review the [upgrade guide](https://docs.goauthentik.io/install-config/upgrade/), and be prepared to adapt environment variables, services, ports, volumes, or other compose-level changes as needed.

```bash
podman compose pull
podman compose up -d
```

Keep an eye on the [Authentik release notes](https://docs.goauthentik.io/releases/) and release-specific notes before moving between versions, especially for major or skipped releases.

## Backups

At minimum, back up these paths and volumes:

- `data/`
- `config/custom-templates/`
- `config/certs/` if you store any local certificates here
- the `database` named volume declared by `compose.yml`

## References

- [Authentik Docker Compose installation](https://docs.goauthentik.io/install-config/install/docker-compose/)
- [Authentik first steps](https://docs.goauthentik.io/install-config/first-steps/)
- [Authentik email configuration](https://docs.goauthentik.io/install-config/email/)
- [Authentik reverse proxy configuration](https://docs.goauthentik.io/install-config/reverse-proxy/)
- [Authentik upgrade guide](https://docs.goauthentik.io/install-config/upgrade/)
