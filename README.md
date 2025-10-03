<!--
SPDX-FileCopyrightText: 2025 Anthony Accioly <anthony@accioly.dev>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# containers

Repo for Podman / Compose / Quadlets

## What is Podman?

[Podman](https://podman.io/) is a daemonless container engine for developing, managing, and running OCI Containers on your Linux system. Unlike Docker, Podman doesn't require a daemon to be running and can run containers rootless, making it more secure and easier to integrate with systemd.

## What is Compose?

[Compose](https://docs.docker.com/compose/) is a tool for defining and running multi-container applications. With Compose, you use a YAML file to configure your application's services, networks, and volumes. Then, with a single command, you can create and start all the services from your configuration.

Podman supports the Compose specification through the `podman compose` command (or the standalone `podman-compose` tool).

### Installing Podman Compose

- For general installation instructions, see the [podman-compose installation guide](https://github.com/containers/podman-compose#installation)
- For Podman Desktop users, follow [these instructions to set up compose](https://podman-desktop.io/docs/compose/setting-up-compose)

## Usage

This repository contains several subdirectories, each with its own `compose.yml` file and configuration. To use any of the services:

1. Navigate to the desired subdirectory:
   ```bash
   cd <subdirectory-name>
   ```

2. Start the services with Podman Compose:
   ```bash
   podman compose up
   ```

3. To run in detached mode (background):
   ```bash
   podman compose up -d
   ```

4. To stop the services:
   ```bash
   podman compose down
   ```

## Available Services

- **cloudflared-tunnel**: Cloudflare Tunnel configuration
- **thelounge**: The Lounge web IRC client

## License

This repository is licensed under the [GNU Affero General Public License v3.0 or later (AGPL-3.0-or-later)](LICENSES/AGPL-3.0-or-later.txt).
