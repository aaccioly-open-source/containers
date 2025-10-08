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

### Environment Variables

- **`OIDENT_PORT`** - The port to expose the service on (default: `113`)
- **`IDENT_USERNAME`** - The username to return for all ident requests when forwarding fails or no specific configuration is set (default: `unknown`)

### Port Configuration

The default port is 113 (standard ident port). You can change this by setting the `OIDENT_PORT` environment variable in a `.env` file.

### Custom Configuration File (Optional)

You can optionally mount a custom `oidentd.conf` file into the container to override the default configuration. This allows you to:

- Forward ident requests to other services
- Configure port-specific behavior
- Customize spoofing and reply rules

To use a custom configuration:

1. Create your `oidentd.conf` file in the same directory
2. Uncomment the volume mount in `compose.yml`:
   ```yaml
   volumes:
     - ./oidentd.conf:/etc/oidentd.conf:ro
   ```
3. Restart the service:
   ```bash
   podman compose up -d
   ```

See [`oidentd.conf(5)`](https://linux.die.net/man/5/oidentd.conf) man page for detailed configuration options.

## Network

This service connects to the `cloudflare-tunnel` network to work with other containerized services that require ident support.
