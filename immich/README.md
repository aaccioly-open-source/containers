<!--
SPDX-FileCopyrightText: 2026 Anthony Accioly <anthony@accioly.dev>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Immich

This configuration runs Immich with PostgreSQL (VectorChord/pgvecto.rs), Valkey (a Redis-compatible alternative), and machine learning services using Podman Compose.

## About

[Immich](https://immich.app/) is a high-performance, self-hosted photo and video backup solution.

This stack is based on the official Immich Docker Compose deployment model and includes:

- **immich-server** — Main API and web application
- **immich-machine-learning** — Face recognition, smart search, OCR, and duplicate detection
- **postgres** — Database with vector extension support for search features
- **valkey** — Redis-compatible queue/cache backend for background jobs

The service is also pre-wired for the `nginx-proxy` setup in this repository through `VIRTUAL_HOST` and `LETSENCRYPT_HOST` environment variables.

## Prerequisites

1. Podman and Podman Compose installed
2. The external `nginx-proxy` network available (if using the reverse proxy)
3. Sufficient storage for media library and database data

## Setup

1. Copy the example environment file:
   ```bash
   cp env.example .env
   ```

2. Edit `.env` and set at least:
   ```bash
   # Media and database storage paths
   UPLOAD_LOCATION=./library
   DB_DATA_LOCATION=./postgres

   # Use a strong alphanumeric password
   DB_PASSWORD=change-me

   # Reverse proxy hostnames
   VIRTUAL_HOST=immich.example.com
   LETSENCRYPT_HOST=immich.example.com
   ```

3. (Recommended) Keep this compose file aligned with Immich official release templates:
   ```bash
   https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
   ```

4. Start Immich:
   ```bash
   podman compose up -d
   ```

5. Open Immich:
   ```text
   http://localhost:2283
   ```
   or your configured domain behind `nginx-proxy`.

## Optional Configuration File

The custom Immich config file (`config/immich-config.json`) is optional and intentionally gitignored in this repository.

If you want to use it, follow the official guide to generate/download a config file:

- https://docs.immich.app/install/config-file

Save it as:

```text
config/immich-config.json
```

To use a custom server config file, uncomment the config volume line in `compose.yml` under `immich-server`:

```yaml
# - ./config/immich-config.json:${IMMICH_CONFIG_FILE}
```

Then confirm `IMMICH_CONFIG_FILE` is set in `.env` (default in `env.example`):

```bash
IMMICH_CONFIG_FILE=/config/immich-config.json
```

## Hardware Acceleration

Hardware acceleration is available for:

- Video transcoding (`immich-server` via `hwaccel.transcoding.yml`)
- Machine learning inference (`immich-machine-learning` via `hwaccel.ml.yml` or image tag suffixes)

See the official docs for platform-specific configuration:

- [Transcoding acceleration](https://docs.immich.app/features/hardware-transcoding)
- [ML acceleration](https://docs.immich.app/features/ml-hardware-acceleration)

## Usage

View logs:
```bash
podman compose logs -f
```

Stop Immich:
```bash
podman compose down
```

Update Immich images and restart:
```bash
podman compose pull
podman compose up -d
```

## Data Persistence

Persistent data is stored in:

- `./library` (or `UPLOAD_LOCATION`) — uploaded media, thumbnails, encoded assets, metadata side files
- `./postgres` (or `DB_DATA_LOCATION`) — PostgreSQL data
- `model-cache` named volume — machine learning model cache

## Network

This service expects the external `nginx-proxy` network:

```yaml
networks:
  nginx-proxy:
    external: true
```

## References

- [Immich Docker Compose installation guide](https://docs.immich.app/install/docker-compose/)
- [Immich environment variables](https://docs.immich.app/install/environment-variables)
- [Immich post-install steps](https://docs.immich.app/install/post-install)
- [Immich upgrade guide](https://docs.immich.app/install/upgrading)
