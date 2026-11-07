# 8. Security and backups

## Basic rules

- Change all default passwords (qBittorrent, File Browser) and use strong unique ones for Portainer, Jellyfin, Uptime Kuma and the Mint user.
- Never forward 9090, 9443, 22 or 8080 on your router. For access away from home use a VPN such as Tailscale.
- Containers run as your user (UID 1000) wherever possible. Portainer has the Docker socket, which is effectively root, so protect its password.
- Membership in the `docker` group is also powerful. Only add trusted users.

## Firewall with UFW (written, not part of the tested build)

Find your subnet with `ip route | grep src` (for example `192.168.1.0/24`). Allow SSH first so you do not lock yourself out:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
for p in 22 445 9090 9443 8080 8081 8096 3001 6881; do sudo ufw allow from 192.168.1.0/24 to any port $p; done
sudo ufw enable
sudo ufw status verbose
```

Docker publishes ports by editing iptables directly, which can bypass UFW. The real protection is that the router has no port forwarding. Re-test every app and SSH after enabling.

## Updates (monthly)

```bash
sudo apt update && sudo apt upgrade -y
cd /mnt/hdd/docker && docker compose pull && docker compose up -d && docker image prune -f
```

## Backups

Two kinds, because they protect different things.

**1. App settings and compose file (small):**

```bash
sudo tar czf /mnt/hdd/backups/appdata-$(date +%F).tar.gz -C /mnt/hdd appdata docker/compose.yaml
```

It sits on the same disk, so copy it off now and then: `scp <user>@<server-ip>:/mnt/hdd/backups/*.tar.gz .`

**2. System snapshots with Timeshift (Mint menu):**

1. Snapshot type **RSYNC** (BTRFS is greyed out on ext4).
2. Location: the HDD partition (`sda2`), not the SSD. Snapshots are stored in `/timeshift`, so do not delete that folder.
3. Levels: Monthly, keep 2.
4. Click **Create** for the first snapshot.

Timeshift protects the Mint system on the SSD (roll back after a bad update). It does not replace the app settings backup.

## Samba (optional, not part of the tested build)

File Browser already covers remote file access. Samba adds `\\<server-ip>\hdd` in Windows Explorer and `smb://<server-ip>/hdd` on Linux:

```bash
sudo apt install -y samba
sudo smbpasswd -a $USER
sudo tee -a /etc/samba/smb.conf > /dev/null <<EOF2

[hdd]
   path = /mnt/hdd
   read only = no
   valid users = $USER
EOF2
testparm -s
sudo systemctl enable --now smbd && sudo systemctl restart smbd
```
