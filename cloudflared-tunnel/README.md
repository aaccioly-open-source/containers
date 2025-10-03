<!--
SPDX-FileCopyrightText: 2025 Anthony Accioly <anthony@accioly.dev>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Cloudflare Tunnel

This configuration runs a Cloudflare Tunnel using Podman Compose.

## Prerequisites

1. A Cloudflare account with Zero Trust enabled
2. A tunnel created in the Cloudflare Zero Trust dashboard
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

## Configuration

The tunnel token is configured via the `TUNNEL_TOKEN` environment variable. Get this from your Cloudflare Zero Trust dashboard when you create a tunnel.
