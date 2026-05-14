# Self-Hosted

Personal home-lab docker stacks for a Proxmox LXC: media acquisition over VPN,
Plex playback, Immich photo management, and Tailscale for off-LAN access.

Each stack lives in its own folder with its own `docker-compose.yml` and `.env`,
so they can be brought up and down independently.

## Stacks

| Folder | What it runs | Key endpoints |
| --- | --- | --- |
| [`media-torrent-vpn/`](media-torrent-vpn/) | gluetun (NordVPN) + qBittorrent + Prowlarr + Sonarr + Radarr — all egress forced through the VPN tunnel | qBittorrent `:8080`, Prowlarr `:9696`, Radarr `:7878`, Sonarr `:8989` |
| [`plex/`](plex/) | Plex media server (host network), reads from `/mnt/media/library` | `:32400` |
| [`immich/`](immich/) | Immich server + machine learning + redis + postgres for self-hosted photos | Web `:2283`, ML `:3003` |
| [`tailscale/`](tailscale/) | Tailscale daemon (host network) so the above are reachable from outside the LAN | — |

## Prerequisites

- Proxmox LXC must allow the `tun` device through for gluetun and tailscale.
  Edit `/etc/pve/lxc/<id>.conf`:
  ```
  lxc.cgroup2.devices.allow: c 10:200 rwm
  lxc.mount.entry: /dev/net dev/net none bind,create=dir
  lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
  ```
- Docker + Docker Compose v2 installed inside the LXC.
- A shared media root at `/mnt/media` (movies, TV, downloads) for qBittorrent / Sonarr / Radarr / Plex.
- `PUID` / `PGID` of the host user that owns `/mnt/media`, set in each stack's `.env`.

## Bring a stack up

```bash
cd <stack>/
cp .env.example .env          # then edit secrets
docker compose up -d
docker compose logs -f
```

To stop:

```bash
docker compose down
```

## Verify the VPN tunnel (media-torrent-vpn)

```bash
docker exec -it gluetun wget -qO- https://ipinfo.io
# or, from any host:
docker run --rm --network=container:gluetun alpine:3.18 sh -c "apk add wget && wget -qO- https://ipinfo.io"
```

The reported IP should be the VPN exit, not your home IP.

## Layout

```
.
├── media-torrent-vpn/   # gluetun + arr stack (see folder README)
├── plex/                # Plex
├── immich/              # Immich photo server
├── tailscale/           # Tailscale node
└── LICENSE
```

## Links

- Gluetun wiki: https://github.com/qdm12/gluetun-wiki
- Immich docs: https://docs.immich.app
- Tailscale docs: https://tailscale.com/kb
- linuxserver.io images: https://docs.linuxserver.io
