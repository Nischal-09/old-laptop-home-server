# 2. Install Linux Mint

## Create the installer USB

1. Download **Linux Mint Cinnamon 64-bit** from <https://linuxmint.com/download.php> (22.3 "Zena" used here, ~2.9 GB).
2. Flash it to a USB stick (8 GB+, it gets erased).
   - **Rufus** (portable) worked reliably: choose **ISO mode**, and click **Yes** when it offers to download the matching Grub file.
   - balenaEtcher froze with greyed-out buttons on this machine. If that happens, use Rufus.

## Boot the USB

Plug it in, restart, tap **F12** at the Dell logo, and pick the USB. Choose **Start Linux Mint 64-bit**. If the screen goes black (NVIDIA GPU), use compatibility mode instead.

## Installer choices

1. Language, keyboard, Wi-Fi.
2. Tick **Install multimedia codecs**, but **untick Configure Secure Boot**. Leaving it on forces a blue MOK enrollment screen on first boot, a common cause of failed boots. NVIDIA drivers can be added later if wanted (a server does not need them).
3. **Installation type -> Something else.**
   - Do not choose "Erase disk": it wipes **every** disk including your HDD.
   - Do not choose "Install alongside": that is dual boot.
4. Select the **SSD row** (`/dev/nvme0n1`, check the model name) -> **New Partition Table** -> Continue. Only the SSD is wiped.
5. Create two partitions in the free space:
   - **512 MB**, Primary, beginning, Use as **EFI System Partition**
   - Remaining space, Primary, Use as **Ext4**, mount point **/**
6. Bootloader device: `/dev/nvme0n1`.
7. Click **Install Now**. The "Write the changes to disks" popup must list **only `/dev/nvme0n1`**. If it mentions `sda`, click Go Back.
8. Timezone, username, strong password. Leave **Encrypt home folder** off (lockout risk on a server) and "Require my password to log in" on.
9. Restart, remove the USB, press Enter.

A Dell logo followed by the Mint logo means it booted from the SSD. If Windows or an error appears instead, the install likely never reached the SSD: boot the USB again and compare `lsblk -f` and `sudo efibootmgr -v` (look for an `ubuntu` entry).

## First update

```bash
sudo apt update && sudo apt upgrade -y
```

Reboot afterward to apply kernel updates.
