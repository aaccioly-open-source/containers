<!--
SPDX-FileCopyrightText: 2025 Anthony Accioly <anthony@accioly.dev>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# ClipCascade

This configuration runs ClipCascade, a self-hosted clipboard sharing service, using Podman Compose.

## About

[ClipCascade](https://github.com/Sathvik-Rao/ClipCascade) is a lightweight, self-hosted clipboard sharing service that allows you to share text and files across devices.

## Prerequisites

- Podman and Podman Compose installed
- The `cloudflare-tunnel` network must be created (see the cloudflared-tunnel folder)

## Setup

1. Copy the example environment file:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` to configure the service:
   ```bash
   # Port to expose (default: 8080)
   CLIPCASCADE_PORT=8080

   # Maximum message size in MiB (default: 1)
   CC_MAX_MESSAGE_SIZE_IN_MiB=1

   # Enable/disable P2P mode (default: false)
   CC_P2P_ENABLED=false

   # Enable/disable user registration (default: true)
   CC_SIGNUP_ENABLED=true

   # Optional: Set allowed CORS origins
   # CC_ALLOWED_ORIGINS=https://clipcascade.example.com
   ```

3. Start ClipCascade:
   ```bash
   podman compose up -d
   ```

4. Access ClipCascade at `http://localhost:8080` (or the port you configured)

## Usage

View logs:
```bash
podman compose logs -f
```

Stop the service:
```bash
podman compose down
```

## Configuration

### Environment Variables

- **`CLIPCASCADE_PORT`** - The port to expose the service on (default: `8080`)
- **`CC_MAX_MESSAGE_SIZE_IN_MiB`** - Maximum message size in MiB, ignored if P2P mode is enabled (default: `1`)
- **`CC_P2P_ENABLED`** - Enables or disables peer-to-peer mode (default: `false`)
- **`CC_ALLOWED_ORIGINS`** - Defines allowed CORS origins for security. Set this if using a reverse proxy or custom domain
- **`CC_SIGNUP_ENABLED`** - Enables or disables user self-registration (default: `true`)

### P2P Mode

When P2P mode is enabled, ClipCascade uses peer-to-peer connections for sharing, which bypasses the message size limit set by `CC_MAX_MESSAGE_SIZE_IN_MiB`.

### CORS Configuration

If you're accessing ClipCascade through a reverse proxy or custom domain, set `CC_ALLOWED_ORIGINS` to match your domain:

```bash
CC_ALLOWED_ORIGINS=https://clipcascade.example.com
```

## Data Persistence

User data and clipboard history are stored in the `./data` directory, so they persist across container restarts.

## Network

This service connects to the `cloudflare-tunnel` network to work with other containerized services and be accessible through Cloudflare Tunnel.

## Security Considerations

- If exposing ClipCascade to the internet, consider:
  - Setting `CC_SIGNUP_ENABLED=false` after creating your accounts
  - Configuring `CC_ALLOWED_ORIGINS` to restrict access
  - Using authentication and strong passwords
  - Placing it behind a reverse proxy with additional security measures
