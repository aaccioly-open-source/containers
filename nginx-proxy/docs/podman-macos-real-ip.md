<!--
SPDX-FileCopyrightText: 2026 Anthony Accioly <anthony@accioly.dev>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Preserving Real Client IPs on macOS with Podman Desktop

## The problem

On macOS, Podman runs containers inside a Linux VM. The VM's networking layer ([gvproxy](https://github.com/containers/gvisor-tap-vsock)) performs NAT on all inbound traffic, replacing the real client IP with `192.168.127.1` — the VM gateway. This is a fundamental limitation: gvproxy does not support source IP preservation or PROXY protocol.

As a result, nginx logs every request as coming from `192.168.127.1`, and any `X-Real-IP` or `X-Forwarded-For` headers sent to upstream services contain the same gateway address instead of the actual client IP.

## The solution

Run **HAProxy on the macOS host** as a TCP front-end. HAProxy sees the real client IP (before gvproxy NATs it) and injects a [PROXY protocol v2](https://www.haproxy.org/download/2.9/doc/proxy-protocol.txt) header into the TCP stream. Since gvproxy only rewrites IP/TCP headers (not payload), the PROXY protocol header survives the NAT and nginx can extract the real IP from it.

```
Client (real IP)
  → macOS:80/443  (HAProxy — sees real IP, prepends PROXY protocol header)
  → macOS:8080/8443  (gvproxy — NATs IP headers, but PROXY header is in TCP payload)
  → Linux VM → nginx container  (reads real IP from PROXY protocol header)
```

## Setup

### 1. Install HAProxy

```bash
brew install haproxy
```

### 2. Configure `.env`

Bind the nginx container to localhost on alternate ports so HAProxy can own ports 80/443, and enable PROXY protocol:

```bash
BIND_ADDRESS=127.0.0.1
HTTP_PORT=8080
HTTPS_PORT=8443
ENABLE_PROXY_PROTOCOL=true
```

### 3. Enable the real IP conf

In `compose.yml`, uncomment the `real_ip.conf` volume mount in the `nginx` service:

```yaml
      # - ./config/conf.d/real_ip.conf:/etc/nginx/conf.d/real_ip.conf:ro
```
→
```yaml
      - ./config/conf.d/real_ip.conf:/etc/nginx/conf.d/real_ip.conf:ro
```

This mounts `config/conf.d/real_ip.conf` (proxy-wide nginx config) that tells nginx to resolve `$remote_addr` from the PROXY protocol header instead of the TCP source address.

### 4. Start HAProxy

```bash
# Test the configuration
haproxy -f config/haproxy.cfg -c

# Run in foreground (for testing)
haproxy -f config/haproxy.cfg

# Or run as a brew service
ln -sf "$(pwd)/config/haproxy.cfg" "$(brew --prefix)/etc/haproxy.cfg"
brew services start haproxy
```

> [!TIP]
> When running as a Homebrew service, HAProxy logs are written to:
>
> ```bash
> $(brew --prefix)/var/log/haproxy.log
> ```
>
> Useful commands:
>
> ```bash
> # Follow logs live
> tail -f "$(brew --prefix)/var/log/haproxy.log"
>
> # Show recent lines
> tail -n 200 "$(brew --prefix)/var/log/haproxy.log"
> ```

### 5. Restart the compose stack

```bash
podman compose down && podman compose up -d
```

## How it works

| Component          | File                         | Role |
|--------------------|------------------------------|------|
| HAProxy            | `config/haproxy.cfg`         | TCP proxy on ports 80/443. Prepends PROXY protocol v2 header and forwards to `127.0.0.1:8080/8443`.|
| nginx real_ip conf | `config/conf.d/real_ip.conf` | `set_real_ip_from` + `real_ip_header proxy_protocol` — proxy-wide nginx config that overrides `$remote_addr` with the real IP from the PROXY protocol header. |
| docker-gen         | `compose.yml`                | `ENABLE_PROXY_PROTOCOL=true` makes the nginx template add `proxy_protocol` to all `listen` directives. |

### PROXY protocol flow in detail

1. A client connects to HAProxy on port 443 (TCP).
2. HAProxy opens a connection to `127.0.0.1:8443` and prepends a PROXY protocol v2 binary header containing the client's real IP and port.
3. gvproxy forwards the connection from the macOS host into the Linux VM via its userspace network stack. It NATs the source IP to `192.168.127.1`, but the PROXY protocol header is part of the TCP payload — untouched.
4. nginx receives the connection. Because `ENABLE_PROXY_PROTOCOL=true` adds `proxy_protocol` to the `listen` directive, nginx parses the PROXY protocol header and populates `$proxy_protocol_addr` with the real client IP.
5. `config/conf.d/real_ip.conf` (mounted as a proxy-wide nginx conf) uses `set_real_ip_from` (trusting private ranges) + `real_ip_header proxy_protocol` to override `$remote_addr` with the real IP from the PROXY protocol header.
6. The standard `proxy_set_header X-Real-IP $remote_addr` and `proxy_set_header X-Forwarded-For ...` directives now carry the real client IP to upstream services.

## Skipping HAProxy (Linux / Docker)

On Linux or Docker where gvproxy is not involved, the real client IP is naturally preserved. Skip HAProxy entirely: leave the `real_ip.conf` mount commented out in `compose.yml` and set in `.env`:

```bash
BIND_ADDRESS=0.0.0.0
HTTP_PORT=80
HTTPS_PORT=443
ENABLE_PROXY_PROTOCOL=false
```

## Further reading

- [podman-networking-docs](https://github.com/eriksjolund/podman-networking-docs) — Comprehensive guide to rootless Podman networking, source IP preservation, and PROXY protocol
- [HAProxy PROXY protocol spec](https://www.haproxy.org/download/2.9/doc/proxy-protocol.txt)
- [nginx-proxy PROXY protocol support](https://github.com/nginx-proxy/nginx-proxy/tree/main/docs#proxy-protocol)
