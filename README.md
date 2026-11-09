# fedora server config

self hosting some stuff, all custom, and all [nord theme](https://github.com/nordtheme/nord)

```
.
├── caddy        reverse proxy + https
├── couchdb      obsidian livesync
├── duckdns      dynamic dns
├── forgejo      git
├── jellyfin     media
└── qbittorrent  torrents through protonvpn
```

each folder has a `compose.yml`, secrets go in a `.env` next to it (copied from `.env.example`). anything the containers create inside these folders is gitignored.

## pre-install

### ignore lid closing

```
sudo mkdir -p /etc/systemd/logind.conf.d
printf '[Login]\nHandleLidSwitch=ignore\n' | sudo tee /etc/systemd/logind.conf.d/lid.conf
```

### hostname

```
sudo hostnamectl set-hostname fedora-server
```

### disable SElinux, so the containers can use the bind mounts - i will find better solution later

```
sudo grubby --update-kernel ALL --args="selinux=0"
sudo reboot
```

## install

```
sudo dnf install neovim git -y
git clone https://github.com/hcg-leo/fedora-server ~/fedora-server
```

### docker

```
sudo dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl enable --now docker
```

### duckdns

```
cd ~/fedora-server/duckdns
cp .env.example .env && nvim .env
sudo docker compose up -d
```

then forward ports 80 and 443 to the server on the router at `http://192.168.1.254/`

### caddy

```
cd ~/fedora-server/caddy
sudo docker compose up -d
```

### jellyfin

make the media folders yourself first, otherwise docker makes them as root and qbittorrent can't write to them

```
cd ~/fedora-server/jellyfin
mkdir -p media/admin/{movies,shows,music} media/family/movies
sudo docker compose up -d
```

> to add a user, make a `media/<user>/...` folder and mount it in `compose.yml`

### qbittorrent

```
cd ~/fedora-server/qbittorrent
cp .env.example .env && nvim .env
sudo docker compose up -d
```

the temporary web ui password is in `sudo docker logs qbittorrent`. in the web ui go to settings -> web ui -> authentication and tick `bypass authentication for clients on localhost`, so gluetun can set the forwarded port and `tun0` interface itself

test at `https://ipleak.net/` - check it matches the ip in `sudo docker logs gluetun`

### forgejo

```
cd ~/fedora-server/forgejo
mkdir -p forgejo/gitea/conf && cp app.ini.example forgejo/gitea/conf/app.ini
sudo docker compose up -d
sudo docker exec -u git forgejo forgejo admin user create --admin --username hcg-leo --password 'password' --email 'email'
```

then go to `https://forgejo.hcg-leo.duckdns.org/user/settings`

for an "open with zed" button, go to `https://forgejo.hcg-leo.duckdns.org/admin/config/settings` and add

```
zed = zed://git/clone?repo={url}
```

### couchdb for obsidian livesync

```
cd ~/fedora-server/couchdb
cp .env.example .env && nvim .env
sudo docker compose up -d
```
