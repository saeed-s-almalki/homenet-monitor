# 02 — Deploying NetAlertX & Pi-hole with Docker

All commands run on the Ubuntu server (`homenet-monitor`, 192.168.0.155).

## 1. Base system

```bash
sudo apt-get update && sudo apt-get -y upgrade
sudo apt-get install -y ca-certificates curl gnupg net-tools arp-scan
```

## 2. Docker Engine + Compose

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker "$USER"     # log out/in to take effect
sudo systemctl enable --now docker
sudo docker run --rm hello-world     # sanity check
```

## 3. NetAlertX

```bash
mkdir -p ~/homenet-monitor && cd ~/homenet-monitor
# copy compose/netalertx/docker-compose.yml here
sudo docker compose up -d
```

Set the ARP-flux sysctls on the **host** (host networking can't set them from compose):

```bash
echo -e "net.ipv4.conf.all.arp_ignore=1\nnet.ipv4.conf.all.arp_announce=2" \
  | sudo tee /etc/sysctl.d/99-netalertx.conf
sudo sysctl -p /etc/sysctl.d/99-netalertx.conf
```

Open `http://192.168.0.155:20211`. The subnet (`192.168.0.0/24 --interface=eth0`) is auto-detected; the first ARP scan runs within ~2 minutes and populates the device list. Rename/classify devices from the UI (TV, AC, Laptop, …).

> **Gotcha:** current NetAlertX images persist to **`/data/config`** and **`/data/db`** (older guides mount `/app/...`). Mounting the wrong paths makes the startup checks fail with exit code 126.

## 4. Free port 53 for Pi-hole

Ubuntu's `systemd-resolved` binds `127.0.0.53:53`. Disable its stub listener and point the host at the real upstream:

```bash
sudo cp /etc/systemd/resolved.conf /etc/systemd/resolved.conf.bak
sudo sed -i 's/^#\?DNSStubListener=.*/DNSStubListener=no/' /etc/systemd/resolved.conf
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
sudo systemctl restart systemd-resolved
sudo ss -tulnp | grep ':53' || echo "PORT 53 FREE"
```

## 5. Pi-hole

```bash
cd ~/homenet-monitor/pihole
cp .env.example .env      # then edit .env and set PIHOLE_PASSWORD
sudo docker compose up -d
```

Verify:

```bash
# normal resolution
nslookup github.com 127.0.0.1
# a known ad domain should return 0.0.0.0 (blocked)
nslookup doubleclick.net 127.0.0.1
```

Admin UI: `http://192.168.0.155:8080/admin`.
