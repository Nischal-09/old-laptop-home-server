# 7. Power and background operation

## Keep running with the lid closed

```bash
sudo mkdir -p /etc/systemd/logind.conf.d
sudo tee /etc/systemd/logind.conf.d/lid.conf <<EOF2
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
EOF2
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
systemctl is-enabled docker     # enabled
```

- The file tells Linux to ignore the lid. `mask` blocks suspend and hibernate completely.
- Also set Cinnamon -> **Power Management**: suspend when inactive = **Never** (A/C and battery). The lid dropdown may be blank because the system file overrides it. Choosing "Do nothing" if offered is fine.
- Do not restart `systemd-logind` in a graphical session (it can log you out). Reboot instead.

## Start automatically after boot

Docker is enabled at boot, and each container has `restart: unless-stopped`. Test:

```bash
sudo reboot
# wait ~2 minutes, then
ssh <user>@<server-ip>
docker ps          # all containers Up
df -h /mnt/hdd     # HDD mounted
```

You do not need to log in on the laptop itself. This requires Wi-Fi to connect without a login (use Ethernet if it does not).

## Lid test

Close the lid for 1-2 minutes and open `http://<server-ip>:8096` from another device. It should still load.

## Power loss, battery and heat

- With **Wake on AC** on in BIOS, the laptop powers on by itself after a blackout, and Docker and the apps come back. A plain power cut still ends the session like any PC, so keep backups.
- The battery acts as a small UPS. Limiting charge to 50-80% extends its life.
- Keep vents clear, especially lid closed. Check temperatures:
  ```bash
  sudo apt install -y lm-sensors && sensors
  ```
  Under about 80 C at idle or light load is fine.
- A laptop uses far less electricity than a desktop, but it still runs 24/7. Estimate roughly 10-25 W at idle with disks spinning.
