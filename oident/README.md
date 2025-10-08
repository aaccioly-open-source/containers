<!--
SPDX-FileCopyrightText: 2025 Anthony Accioly <anthony@accioly.dev>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# oidentd

This configuration runs oidentd (an ident daemon) in a lightweight Alpine Linux container using Podman Compose.

## Prerequisites

- Podman and Podman Compose installed
- The `cloudflare-tunnel` network must be created (see the cloudflared-tunnel folder)

## Setup

1. (Optional) Create a `.env` file to customize the port:
   ```bash
   echo "OIDENT_PORT=113" > .env
   ```

2. Build and start the service:
   ```bash
   podman compose up -d --build
   ```

## Usage

View logs:
```bash
podman compose logs -f
```

Stop the service:
```bash
podman compose down
```

Rebuild after configuration changes:
```bash
podman compose up -d --build
```

## Configuration

The oidentd configuration is stored in `oidentd.conf` and mounted into the container. By default, it allows users to use the ident service with various options including spoofing and random responses.

Edit `oidentd.conf` to customize the behavior. See `oidentd.conf(5)` for more information on available options.

### Port Configuration

The default port is 113 (standard ident port). You can change this by setting the `OIDENT_PORT` environment variable in a `.env` file.

## Network

This service connects to the `cloudflare-tunnel` network to work with other containerized services that require ident support.

## Image Optimization

The image is based on Alpine Linux for minimal size and uses the official oidentd package. The daemon runs in foreground mode (`-i -S -f` flags) suitable for container execution.
