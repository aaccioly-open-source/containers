<!--
SPDX-FileCopyrightText: 2025 Anthony Accioly <anthony@accioly.dev>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Cloudflare Tunnel

This configuration runs a Cloudflare Tunnel using Podman Compose.

## Prerequisites

1. A Cloudflare account with Zero Trust enabled
2. A tunnel created in the Cloudflare Zero Trust dashboard (see [Cloudflare Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/) for guidance)
3. A tunnel token from your Cloudflare tunnel configuration

## Setup

1. Copy the example environment file:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` and add your tunnel token:
   ```
   TUNNEL_TOKEN=your-actual-tunnel-token
   ```

3. Start the tunnel:
   ```bash
   podman compose up -d
   ```

## Usage

View logs:
```bash
podman compose logs -f
```

Stop the tunnel:
```bash
podman compose down
```

> [!NOTE]
> `podman compose down` also removes the network created for the tunnel if no other containers are connected to it.
> Make sure to detach any other containers from the `cloudflare-tunnel` network before stopping the tunnel.

## Configuration

The tunnel token is configured via the `TUNNEL_TOKEN` environment variable. Get this from your Cloudflare Zero Trust dashboard when you create a tunnel.

## Connecting Other Containers

Other containers need to join the `cloudflare-tunnel` network to be accessible through the tunnel. This can be done in two ways:

### Using Podman CLI

When running a container directly with podman, use the `--network` flag:

```bash
podman run -d \
  --name my-app \
  --network cloudflare-tunnel \
  -p 8080:8080 \
  my-app-image
```

To connect an already running container to the network:

```bash
podman network connect cloudflare-tunnel my-app
```

### Using Podman Compose

In your application's `compose.yml`, reference the external network:

```yaml
services:
  my-app:
    container_name: my-app
    image: my-app-image
    ports:
      - "8080:8080"
    networks:
      - cloudflare-tunnel

networks:
  cloudflare-tunnel:
    external: true
```

> [!TIP]
> The `external: true` directive tells Podman Compose that the network already exists and should not be created.
