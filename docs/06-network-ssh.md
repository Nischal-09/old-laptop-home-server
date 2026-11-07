# 6. Network, static IP and SSH

## Local vs internet access

Everything here is reachable only from devices on your home network. Nothing is exposed to the internet as long as your router has no port forwarding.

## Give the server a stable IP

Best option: a **DHCP reservation** in your router (reserve the server's MAC address for its IP). Find the adapter and address:

```bash
ip -br addr                  # line with your IP shows the active adapter (wlp... = Wi-Fi, enp... = Ethernet)
ip route | grep default      # shows gateway, for example 192.168.1.1
ip link                      # MAC address under the adapter
```

Router admin pages use a separate login from the Wi-Fi password (often on a sticker, or from your ISP). Do not factory-reset an ISP router.

### Fallback: static IP on the laptop

If you cannot reach the router, set it in Mint: Network Settings -> your connection -> gear -> IPv4:

- Method: **Manual**
- Address: `192.168.1.200` (pick something outside your router's automatic range)
- Netmask: `24`
- Gateway: your router (for example `192.168.1.1`)
- DNS: turn **Automatic** off and enter `192.168.1.1,1.1.1.1`

Click Apply. The Details tab will not change until the connection restarts:

```bash
nmcli connection show                       # copy the exact connection name
sudo nmcli connection down "<connection-name>"
sudo nmcli connection up "<connection-name>"
hostname -I                                 # should now show the new address
```

Run these on the laptop itself, since the restart drops remote sessions. Afterwards update bookmarks and Uptime Kuma monitors to the new address. (`172.17.0.1`/`172.18.0.1` in `hostname -I` are Docker's internal networks.)

Ethernet is more reliable than Wi-Fi for a server. Each adapter has its own MAC, so reserve the one you will use long term.

## SSH

```bash
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
systemctl is-active ssh        # active
```

From your other computer:

```bash
ssh <user>@<server-ip>
```

Type `yes` at the fingerprint question. Anything you run here runs on the server.

### Optional: key login and hardening (not part of the tested build)

On the **client**:

```bash
ssh-keygen -t ed25519
ssh-copy-id <user>@<server-ip>
ssh <user>@<server-ip>          # must log in without a password
```

Only then, on the server:

```bash
sudo tee /etc/ssh/sshd_config.d/99-hardening.conf <<EOF2
PasswordAuthentication no
PermitRootLogin no
EOF2
sudo sshd -t && sudo systemctl restart ssh
```

Keep your first session open while testing a second one.

## Phone

Join the same Wi-Fi and use the same addresses, for example the Jellyfin app with `http://<server-ip>:8096`.
