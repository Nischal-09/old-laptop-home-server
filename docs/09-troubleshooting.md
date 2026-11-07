# 9. Troubleshooting

Problems below happened on the real build.

| Symptom | Cause | Fix |
|---|---|---|
| Installer or live USB sees no SSD; `ls /dev/nvme*` says no such file | BIOS in RAID On (Intel RST) | BIOS -> Storage -> AHCI, click **APPLY CHANGES**, exit |
| After install, laptop boots Windows | Install never reached the SSD (RAID mode) | Fix AHCI, compare `lsblk -f` and `sudo efibootmgr -v` from the live USB, reinstall |
| balenaEtcher buttons greyed out | Etcher hang | Use Rufus (ISO mode) |
| "Please remove the installation medium" after starting a restart from the live session | Live USB shutting down, not an install finishing | Continue the BIOS/boot steps, do not assume Mint is installed |
| `mkfs.ext4`: "is mounted; will not make a filesystem here" | Partition still mounted, or typo (`unmount`, `dev/sda2`) | `sudo umount /dev/sda2`, copy-paste commands |
| Bad `/etc/fstab` line with old UUID | Command ran before the format | `sudo sed -i '/OLD-UUID/d' /etc/fstab`, check with `tail -5 /etc/fstab` |
| HDD missing from file manager sidebar | fstab mounts under `/mnt` are system mounts | Browse File System -> mnt -> hdd, press Ctrl+D |
| Browser: "DNS address could not be found" | Typed the literal word `ip` | Use the real address from `hostname -I` |
| Portainer "timed out for security purposes" | No admin created in 5 minutes | `docker restart portainer`, create admin quickly |
| Portainer 403 on create user | Stale setup token | Newest token only: `docker logs portainer 2>&1 \| grep setup_token \| tail -1`, reload page |
| Portainer "cannot connect to local environment" | First-start socket miss, or already connected | Check Home first, add via wizard (Docker Standalone -> Socket) only if absent. Remove duplicates |
| Jellyfin wizard "error processing the request", log `CustomAuthentication was forbidden` | Admin created on an earlier attempt, retry rejected | Reset only its config: `docker compose stop jellyfin`, `rm -rf /mnt/hdd/appdata/jellyfin/config/* /mnt/hdd/appdata/jellyfin/cache/*`, `docker compose up -d jellyfin`, rerun wizard once in incognito |
| Jellyfin "Server Mismatch" | Browser remembers old server ID | Connect Anyway |
| Uptime Kuma "version is outdated" | `latest` tag is v1 | Image `louislam/uptime-kuma:2`, clear empty data folder, pull and restart |
| Static IP set but `hostname -I` unchanged | Connection not restarted | `sudo nmcli connection down/up "<name>"` (run on the laptop) |
| Router login rejects Wi-Fi password | Admin login differs | Sticker or ISP. Fall back to static IP on the laptop |

## General diagnostics

```bash
docker ps -a                   # container states
docker logs <name> 2>&1 | tail -30
lsblk -f                       # storage problems
df -h                          # free space
sudo systemctl status docker
```
