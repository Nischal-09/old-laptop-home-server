# 4. Install Docker

## Why Docker

Each app runs in its own sealed container with everything it needs. The system stays clean, apps cannot conflict, updating or removing one is one command, and the whole server is described by one text file. Containers share the host kernel, so they are light enough to run alongside a normal desktop.

## Add Docker's repository

Mint's codename is not known to Docker's repo, so use the Ubuntu base name:

```bash
grep UBUNTU_CODENAME /etc/os-release     # should print noble
```

```bash
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
```

`apt update` should fetch from `download.docker.com` without errors. A harmless `Ign: packages.linuxmint.com ... InRelease` line may appear.

## Install

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
```

The group change lets you run `docker` without `sudo` after a reboot or re-login. Reboot now.

## Move Docker's data to the HDD (do this before pulling images)

```bash
sudo tee /etc/docker/daemon.json <<EOF2
{"data-root":"/mnt/hdd/docker-data","log-driver":"json-file","log-opts":{"max-size":"10m","max-file":"3"}}
EOF2
sudo mkdir -p /etc/systemd/system/docker.service.d
printf "[Unit]\nRequiresMountsFor=/mnt/hdd\n" | sudo tee /etc/systemd/system/docker.service.d/hdd.conf
sudo systemctl daemon-reload && sudo systemctl restart docker
```

- `data-root` stores images and containers on the HDD.
- The log options cap each container at 3 files of 10 MB.
- `RequiresMountsFor` makes Docker wait for the HDD at boot, so it never quietly writes to the SSD.

## Verify

```bash
docker info | grep "Docker Root Dir"     # /mnt/hdd/docker-data
docker run --rm hello-world              # prints "Hello from Docker!"
```
