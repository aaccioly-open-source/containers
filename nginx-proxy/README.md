<!--
SPDX-FileCopyrightText: 2026 Anthony Accioly <anthony@accioly.dev>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Nginx Reverse Proxy

This configuration runs a 3-container nginx reverse proxy with automated SSL certificates via ACME challenges, using Podman Compose.

## About

This setup uses the [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) ecosystem:

- **nginx** — The reverse proxy server
- **[docker-gen](https://github.com/nginx-proxy/docker-gen)** — Watches containers and generates nginx configuration from templates
- **[acme-companion](https://github.com/nginx-proxy/acme-companion)** — Automates SSL/TLS certificate issuance and renewal via Let's Encrypt

Proxied containers are auto-discovered by setting `VIRTUAL_HOST` and `LETSENCRYPT_HOST` environment variables on them.

## Prerequisites

1. A DNS provider supported by [acme.sh](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) (default: Cloudflare)
2. An API token for your DNS provider (for Cloudflare: [create token](https://dash.cloudflare.com/profile/api-tokens) with **Zone - DNS - Edit** permissions)
3. Ports 80 and 443 available on the host

## Setup

1. Copy the example environment file:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` and configure your ACME and DNS settings:
   ```bash
   # ACME challenge type (default: DNS-01)
   ACME_CHALLENGE=DNS-01

   # acme.sh DNS API plugin (default: dns_cf for Cloudflare)
   ACMESH_DNS_API=dns_cf

   # Cloudflare API Token
   CF_TOKEN=your-actual-token
   CF_ACCOUNT_ID=your-account-id
   CF_ZONE_ID=
   ```

   The `compose.yml` builds the `ACMESH_DNS_API_CONFIG` YAML block from these individual variables (configured for Cloudflare by default). For other DNS providers, override `ACMESH_DNS_API_CONFIG` directly with the appropriate keys — see the [acme-companion DNS-01 docs](https://github.com/nginx-proxy/acme-companion/blob/main/docs/Let's-Encrypt-and-ACME.md#dns-01-acme-challenge) and the [acme.sh DNS API wiki](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) for provider-specific configuration.

3. Optionally enable IPv6 support:
   ```bash
   # Enable IPv6 listen directives in generated nginx config
   ENABLE_IPV6=true
   # Prefer IPv6 when connecting to upstream containers
   PREFER_IPV6_NETWORK=true
   ```

4. Download the `nginx.tmpl` template from GitHub:
   ```bash
   curl https://raw.githubusercontent.com/nginx-proxy/nginx-proxy/main/nginx.tmpl > config/nginx.tmpl
   ```

   See [acme-companion advanced usage](https://github.com/nginx-proxy/acme-companion/blob/main/docs/Advanced-usage.md) for related template customization guidance.

5. Start the reverse proxy:
   ```bash
   podman compose up -d
   ```

## Podman Socket

The docker-gen and acme-companion containers need access to the container runtime socket to discover running containers. For Podman, enable the socket service:

```bash
# Rootless
systemctl --user enable --now podman.socket

# Rootful
sudo systemctl enable --now podman.socket
```

Then update `CONTAINER_SOCKET` in `.env` to match your socket path:

| Runtime          | Socket Path                          |
|------------------|--------------------------------------|
| Docker           | `/var/run/docker.sock`               |
| Podman (rootful) | `/run/podman/podman.sock`            |
| Podman (rootless)| `/run/user/<UID>/podman/podman.sock` |

## Exposing a Service

To expose a container through the reverse proxy, add these environment variables to it and connect it to the `nginx-proxy` network:

```yaml
services:
  my-app:
    # ...
    environment:
      VIRTUAL_HOST: app.example.com
      VIRTUAL_PORT: "8080"
      LETSENCRYPT_HOST: app.example.com
    networks:
      - default
      - nginx-proxy

networks:
  nginx-proxy:
    external: true
```

## Per-Virtual-Host Custom Configuration

Custom nginx configuration can be applied per virtual host by adding files to the `config/vhost.d/` directory. The nginx-proxy template supports two levels:

- **Server level** — `config/vhost.d/<VIRTUAL_HOST>` — included in the `server` block
- **Location level** — `config/vhost.d/<VIRTUAL_HOST>_location` — included in the `location` block

For example, `config/vhost.d/immich.accioly.social_location` sets a 50GB upload limit for the Immich photo service:

```nginx
client_max_body_size 50g;
proxy_request_buffering off;
```

See the [nginx-proxy custom configuration docs](https://github.com/nginx-proxy/nginx-proxy/tree/main/docs#custom-nginx-configuration) for more details.

## Usage

View logs:
```bash
podman compose logs -f
```

Stop the reverse proxy:
```bash
podman compose down
```

## Network

This service creates the `nginx-proxy` network. Other services that need to be proxied should join this network by declaring it as external:

```yaml
networks:
  nginx-proxy:
    external: true
```
