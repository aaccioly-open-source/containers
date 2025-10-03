<!--
SPDX-FileCopyrightText: 2025 Anthony Accioly <anthony@accioly.dev>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# The Lounge

This configuration runs The Lounge, a self-hosted web IRC client, using Podman Compose.

## About

[The Lounge](https://thelounge.chat/) is a modern, responsive, cross-platform, self-hosted web IRC client.

## Setup

1. (Optional) Copy the example environment file to customize the port:
   ```bash
   cp .env.example .env
   ```

2. Start The Lounge:
   ```bash
   podman compose up -d
   ```

3. Access The Lounge at `http://localhost:9000` (or the port you configured)

## First Time Setup

When you first access The Lounge, you'll need to create a user account. You can do this by accessing the container:

```bash
podman exec -it thelounge thelounge add <username>
```

Follow the prompts to set a password.

## Usage

View logs:
```bash
podman compose logs -f
```

Stop The Lounge:
```bash
podman compose down
```

## Data Persistence

User data, configuration, and IRC logs are stored in the `./data` directory, so they persist across container restarts.

## Configuration

- `THELOUNGE_PORT`: The port to expose The Lounge on (default: 9000)
- `THELOUNGE_IDENTD_PORT`: The port for the identd server (default: 113)

## Network

This service is configured to join the `cloudflared-tunnel` external network. Make sure this network exists before starting the service. You can create it with:

```bash
podman network create cloudflared-tunnel
```
