# 3. Storage: format and mount the HDD

The goal: system on the SSD, everything big on the HDD.

## Why ext4 instead of NTFS

Windows (NTFS) partitions cannot store Linux ownership and permissions. Docker and its apps then hit "permission denied" errors and run slowly. ext4 is Linux's native format.

## Identify the target partition

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,MODEL
```

Choose the partition you want to convert (here `sda2`, ~492 GiB, NTFS, label DATA). Leave the other partition (`sda3`) alone if it holds files you want to keep.

## Format (ERASES the partition)

Make sure it is not mounted (close file manager windows on it), then:

```bash
sudo umount /dev/sda2
sudo mkfs.ext4 -L HDD /dev/sda2
```

Type `y` at "Proceed anyway?". It ends with "Writing superblocks and filesystem accounting information: done". If it says the device is mounted, `mkfs` refuses to run. That safety check is working, so unmount first. Copy-paste commands; `unmount` or `dev/sda2` (no leading slash) typos fail.

## Mount permanently

```bash
sudo mkdir -p /mnt/hdd
UUID=$(sudo blkid -s UUID -o value /dev/sda2)
echo "UUID=$UUID /mnt/hdd ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
sudo systemctl daemon-reload && sudo mount -a
sudo chown $USER:$USER /mnt/hdd
```

- The fstab line mounts the disk at every boot. `nofail` stops a missing HDD from blocking boot.
- Check the printed UUID is the **new** one. A wrong line (for example the old NTFS UUID) can break boot. Remove it with `sudo sed -i '/OLD-UUID/d' /etc/fstab`.
- `chown` lets your user write to the mount.

## Create the folder layout

```bash
mkdir -p /mnt/hdd/{media/movies,media/tv,downloads,files,backups,docker,docker-data}
mkdir -p /mnt/hdd/appdata/{portainer,filebrowser/database,filebrowser/config,qbittorrent,jellyfin/config,jellyfin/cache,uptime-kuma}
df -h /mnt/hdd
```

`df -h` should show about 484G on `/mnt/hdd`.

## Seeing it in the file manager

fstab mounts under `/mnt` are treated as system mounts, so the drive does not appear under Devices. Open **File System -> mnt -> hdd** and press **Ctrl+D** to bookmark it.

## Linux terms in short

- **Mount point**: the folder where a drive's contents appear (`/mnt/hdd`).
- **Ownership/permissions**: who may read or write a file. `chown` changes the owner.
- **Drive letters do not exist**: the SSD is `/`, the HDD is `/mnt/hdd`.
