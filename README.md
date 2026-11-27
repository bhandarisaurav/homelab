# homelab

Docker Compose stacks for the homelab, running in the `docker` LXC (CT 106, `10.0.0.105`) on Proxmox.
Arcane (`:3552`) manages the stacks from this directory.

## Stacks

| Stack | Port | Notes |
|---|---|---|
| `homepage/` | 80 | Dashboard. Config in `homepage/config/` |
| `immich-app/` | 2283 | Photos. Library + DB on `/HDD/immich` |
| `immichframe/` | 8081 | Photo frame for Immich |
| `dawarich/` | 3000 | Location history |
| `stirling-pdf/` | 8080 | PDF tools |
| `speedtest-tracker/` | 8765 | Internet speed history (SQLite) |
| `arcane/` | 3552 | Docker stack manager |
| `portainer/`, `autoheal/` | — | Not currently deployed |

Other services run in their own LXCs: AdGuard (10.0.0.61), Nginx Proxy Manager (10.0.0.28:81),
Syncthing (10.0.0.49:8384), Reitti (10.0.0.17:8080), Homebox (10.0.0.4:7745), NetBird (CT 100).

## Secrets

Secrets are **not** in git. Each stack that needs them has a git-ignored file plus a committed example:

| Stack | Secret file | Template |
|---|---|---|
| `homepage/` | `.env` | `.env.example` |
| `arcane/` | `.env` | `.env.example` |
| `immich-app/` | `.env` | `.env.example` |
| `dawarich/` | `.env` | `.env.example` |
| `speedtest-tracker/` | `.env` | `.env.example` |
| `immichframe/` | `Config/api.key` | `Config/api.key.example` |

## Recreate on a new machine

1. Install Docker and the NVIDIA container toolkit (Immich ML uses the GPU).
2. Make sure `/HDD` is mounted (Immich library/DB, media).
3. Clone the repo:
   ```sh
   git clone git@github.com:bhandarisaurav/homelab.git /root/homelab && cd /root/homelab
   ```
4. For every stack, create the secret file from its template:
   ```sh
   for d in */; do [ -f "$d.env.example" ] && [ ! -f "$d.env" ] && cp "$d.env.example" "$d.env"; done
   ```
   Then fill in the values. Use the same `ARCANE_ENCRYPTION_KEY` and Immich `DB_PASSWORD` as before to keep existing data readable.
5. Create the ImmichFrame key file: `printf '%s' '<immich-api-key>' > immichframe/Config/api.key`
6. Start the stacks:
   ```sh
   for d in immich-app dawarich stirling-pdf immichframe speedtest-tracker homepage arcane; do
     (cd "$d" && docker compose up -d)
   done
   ```
7. Fix ownership where containers run as uid 1000:
   ```sh
   chown -R 1000:1000 immichframe/Config speedtest-tracker/config
   chown -R 1000:991 homepage/config
   ```
   `991` is the docker group; Homepage runs with `PGID=991` so it can read the docker socket.

Data that is **not** in git, so back it up separately: `/HDD/immich`, `dawarich/dawarich_*`, `stirling-pdf/stirling-data`,
`speedtest-tracker/config`, `arcane/data`.
