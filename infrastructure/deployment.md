# Deployment — formularz reklamacji Allegro

> **Status: nothing deployed.** This file describes what this repo will own on the shared
> VPS once a master plan decides it.

## What this repo will own on the host

| Path or resource | Owner | Notes |
|---|---|---|
| `/opt/reklamacje/docker-compose.yml` | **this repo** | Own Compose project and network, every service with a `mem_limit`. |
| `/etc/nginx/conf.d/reklamacje.conf` | **this repo** | One file per vhost. |
| DNS record and TLS certificate for the subdomain | **this repo** | A subdomain, not a path under `agents.skalus.com`, so no file has two writers. |

## What this repo must never touch

The VPS host itself, the Docker engine, certbot's configuration, `/opt/haxe-agents/`,
`/opt/haxe-flow/`, `/opt/book-notes-mcp/`, and their nginx files. Read them freely;
change nothing.

## The memory constraint

8 GB total on the VPS, shared with n8n, Docling (needs ≥ 4 GB free to load), Grafana
and the book-notes MCP. Block 0 measures headroom with `free -h` and `docker stats`
before anything is added.
