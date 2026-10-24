# Media Stack

Docker Compose setup for a self-hosted media server on TrueNAS.

## Services

| Service | Purpose | Port | Network |
|---|---|---|---|
| gluetun | WireGuard VPN, HTTP proxy | 8888, 8388, 8000 | bridge |
| qbittorrent | Torrent client | 8080 | VPN |
| prowlarr | Indexer manager | 9696 | VPN |
| speedtest | Speedtest tracker | 8765 | VPN |
| qbit_manage | qBittorrent housekeeping | – | bridge |
| cross-seed | Cross-seeding | 2468 | bridge |
| unpackerr | Extracts archived downloads | – | bridge |
| sonarr / sonarr4k | TV (HD / 4K) | 8989 | host |
| radarr / radarr4k | Movies (HD / 4K) | 7878 | host |
| bazarr / bazarr4k | Subtitles (HD / 4K) | 6767 / 6768 | bridge |
| recyclarr | Syncs TRaSH guide profiles | – | bridge |
| plex-server | Media server (NVIDIA transcoding) | 32400 | host |
| tautulli | Plex statistics | 8181 | bridge |
| seerr | Media requests | 5055 | bridge |
| autoheal | Restarts unhealthy containers | – | bridge |

The sonarr4k and radarr4k ports are set in each app's own settings.

VPN services share gluetun's network, so their ports are published on **gluetun**, not on the service itself.

## Setup

```sh
cp .env.example .env   # fill in values
docker compose up -d
```

Requires the NVIDIA container runtime for Plex and a WireGuard config at
`${CONFIG}/config/gluetun/wireguard/77.conf`.

## Layout

Shared settings are defined once as YAML anchors at the top of `docker-compose.yml`:

- `x-base`: restart policy and log rotation, used by every service
- `x-env`: `PUID` / `PGID` / `TZ`
- `x-vpn`: routes a service through gluetun
- `x-arr`: shared base for the Sonarr/Radarr instances

Run `docker compose config <service>` to see a service's fully resolved config.

Media lives under `${NFS}/data`, mounted as `/data` in every service that touches files, so hardlinks work.
