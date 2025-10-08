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

1. (Optional) Create a `.env` file to customize the configuration:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` to configure the service:
   ```bash
   # Port to expose (default: 113)
   OIDENT_PORT=113

   # Username to return for ident requests (default: unknown)
   IDENT_USERNAME=myusername
   ```

3. Build and start the service:
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

### Environment Variables

- **`OIDENT_PORT`** - The port to expose the service on (default: `113`)
- **`IDENT_USERNAME`** - The username to return for all ident requests when forwarding fails or no specific configuration is set (default: `unknown`)

### Port Configuration

The default port is 113 (standard ident port). You can change this by setting the `OIDENT_PORT` environment variable in a `.env` file.

## Network

This service connects to the `cloudflare-tunnel` network to work with other containerized services that require ident support.
