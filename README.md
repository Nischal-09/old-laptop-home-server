# Hybrid Linux Mint Desktop + Home Server

Turn an old laptop into a **normal Linux Mint desktop that also runs a home server** in the background: media streaming, downloads, web file manager, monitoring and a web dashboard, all reachable from your other devices. No dual boot, no separate server OS.

Every step in this guide was performed on real hardware. Problems that actually came up are documented in [Troubleshooting](docs/09-troubleshooting.md).

## What you get

| Service | Purpose | Port |
|---|---|---|
| Cockpit | System monitoring and browser terminal | 9090 (https) |
| Portainer CE | Docker web UI | 9443 (https) |
| File Browser | Upload, download, rename files on the HDD | 8080 |
| qBittorrent | Download manager (web UI) | 8081 |
| Jellyfin | Media streaming | 8096 |
| Uptime Kuma | Service availability monitor | 3001 |
| SSH | Remote terminal | 22 |

## Reference hardware

- Dell G3 3500, Intel i5-10300H, 16 GB RAM
- 256 GB NVMe SSD (system) and 1 TB HDD (data)
- Linux Mint 22.3 Cinnamon (Ubuntu 24.04 "noble" base)

Any laptop with 8 GB+ RAM and a separate larger data drive works. The i5 with 16 GB RAM idles at roughly 2.5 GB used with all containers running.

## Architecture

```
SSD (NVMe)  ->  Linux Mint + Docker engine (program files only)
HDD         ->  /mnt/hdd   (ext4)
                 |- docker-data/  Docker images, containers, volumes
                 |- appdata/      per-app config
                 |- downloads/    qBittorrent downloads
                 |- media/        movies, tv (Jellyfin library)
                 |- files/        general files
                 |- backups/      config backups
                 |- docker/       compose.yaml
```

Keeping Docker's data on the HDD stops images, logs and downloads from filling the small SSD.

## Guide

1. [Hardware check and BIOS](docs/01-hardware-and-bios.md)
2. [Install Linux Mint](docs/02-install-mint.md)
3. [Storage: format and mount the HDD](docs/03-storage.md)
4. [Install Docker](docs/04-docker.md)
5. [Deploy the apps](docs/05-apps.md)
6. [Network, static IP and SSH](docs/06-network-ssh.md)
7. [Power and background operation](docs/07-power.md)
8. [Security and backups](docs/08-security-backups.md)
9. [Troubleshooting](docs/09-troubleshooting.md)

The stack definition is in [`compose.yaml`](compose.yaml).

## Quick results check

After a reboot, wait about 2 minutes, then from another device on the same network:

```bash
ssh <user>@<server-ip>
docker ps          # 5 containers should show "Up"
df -h /mnt/hdd     # HDD mounted
```

Open `http://<server-ip>:8096` for Jellyfin, and so on. Nothing needs starting manually.

## Safety rules

- Never port-forward 9090, 9443, 22 or 8080 on your router. For remote access use a VPN such as Tailscale.
- Formatting erases data. Always confirm the partition with `lsblk` first.
- Change every default password.

## Status of each part

Tested end to end: Mint install, HDD setup, Docker, all 5 containers, auto-restart after reboot, static IP, SSH, power settings, Timeshift. Written but not part of the tested build: UFW rules, Samba, SSH key hardening. Test those on your own setup before relying on them.

## License

MIT
