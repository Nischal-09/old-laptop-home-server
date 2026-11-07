# 5. Deploy the apps

## Management: Cockpit

```bash
sudo apt install -y cockpit
sudo systemctl enable --now cockpit.socket
```

Open `https://<server-ip>:9090` and log in with your Mint user. Accept the self-signed certificate warning (Advanced -> Proceed). The **Terminal** tab gives a browser shell, handy for copying text between tabs. A "1 service has failed" notice on a fresh install is usually harmless.

Cockpit covers monitoring, Portainer covers containers. CasaOS was skipped to avoid overlapping dashboards.

## Start the stack

Confirm your IDs match the compose file:

```bash
id -u && id -g      # both should print 1000, otherwise edit the 1000s in compose.yaml
```

Copy [`compose.yaml`](../compose.yaml) to `/mnt/hdd/docker/compose.yaml`, then:

```bash
cd /mnt/hdd/docker && docker compose up -d
docker ps
```

The first run downloads images (a few minutes). Five containers should show **Up**. `restart: unless-stopped` brings them back after every reboot.

Find the server address with `hostname -I` and use it wherever `<server-ip>` appears. Do not type the literal word `ip` in the browser.

## Portainer (`https://<server-ip>:9443`)

- It locks itself if no admin is created within 5 minutes of starting. If you see "timed out for security purposes": `docker restart portainer`.
- Newer versions ask for a **Setup token**. Every restart generates a new one, and only the newest works. Get it with:
  ```bash
  docker logs portainer 2>&1 | grep setup_token | tail -1
  ```
  A 403 error usually means a stale token. Reload the page and use the newest one. Use Cockpit's terminal to copy and paste it between tabs.
- Password must be 12+ characters. At the Edge Compute screen leave it off and click **Skip**.
- If it says it cannot connect to the local environment, the automatic `local` entry may still be Up. Check Home before adding another (a duplicate name is rejected).

## File Browser (`http://<server-ip>:8080`)

Username `admin`. Get the generated password:

```bash
docker logs filebrowser 2>&1 | grep -i password
```

It shows every folder on the HDD. Change the password under Settings -> Profile. Do not delete `docker`, `docker-data` or `appdata`.

## qBittorrent (`http://<server-ip>:8081`)

User `admin`, temporary password from:

```bash
docker logs qbittorrent 2>&1 | grep -i password
```

The temporary password changes on every restart, so set a permanent one (gear icon -> WebUI -> Authentication). Default Save Path must be `/downloads`, which is `/mnt/hdd/downloads`. The status bar showing the HDD's free space confirms it. Only download content you have the right to, for example Linux ISOs.

## Jellyfin (`http://<server-ip>:8096`)

Run the setup wizard once, clicking Next one time per screen:

1. Server name and an admin user.
2. Add library: Movies, folder `/media/movies` (maps to `/mnt/hdd/media/movies`). Set language and region. Unticking Screen Grabber lowers CPU use.
3. Leave **Allow remote connections** on (this is for your home network, not the internet) and UPnP off.

Add movies by uploading to `media/movies` with File Browser, then Dashboard -> Libraries -> Scan. A "Server Mismatch" warning in a browser that remembers an old install is safe: click **Connect Anyway**. Phones: install the Jellyfin app and enter the server address.

## Uptime Kuma (`http://<server-ip>:3001`)

Use the `:2` image tag (already in `compose.yaml`). The `latest` tag still points to unmaintained v1. v1 and v2 databases are not compatible, so choose the tag before creating an account. Add HTTP monitors for 8096, 8080 and 8081. Skip Portainer and Cockpit (self-signed certificates). It reports problems but does not fix them. Restarts come from the Docker restart policy.
