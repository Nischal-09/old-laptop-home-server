# 1. Hardware check and BIOS

## Check the hardware

On Windows: `Ctrl+Shift+Esc` -> Performance tab shows CPU, RAM and disks. On Linux:

```bash
lscpu | grep "Model name"      # processor
free -h                        # RAM
lsblk -o NAME,SIZE,TYPE,MODEL,ROTA   # drives (ROTA 0 = SSD, 1 = HDD)
```

Back up anything you need. Installing and formatting erases drives.

## Switch storage mode to AHCI (important on Intel laptops)

If the BIOS storage mode is **RAID On (Intel RST)**, Linux cannot see an NVMe SSD. The installer will not list it, and `ls /dev/nvme*` returns "No such file or directory".

1. Restart, tap **F2** at the Dell logo to open BIOS.
2. Go to **Storage -> SATA Operation** and select **AHCI**.
3. Click **APPLY CHANGES** (the change counter must show 1), then **EXIT**.

Skipping Apply means the setting is silently not saved. Windows will no longer boot after this, which is expected when Mint replaces it.

Verify later from a live USB:

```bash
lsblk -f     # nvme0n1 must now be listed
```

## Optional BIOS settings for a server

- **System Management -> Wake on AC: ON**: the laptop powers on by itself after a blackout.
- **Power -> Battery Charge Configuration -> Custom (start 50%, stop 80%)**: protects the battery on a laptop that stays plugged in.
