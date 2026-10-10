# fedora server config

_self host!!!_ - self hosting some stuff, all custom, and all [nord theme](https://github.com/nordtheme/nord)!

```
.
├── .gitignore
├── README.md
├── duckdns
│   ├── .env.example
│   └── duckdns.yml
├── forgejo
│   ├── forgejo
│   │   └── gitea
│   │       ├── conf
│   │       │   └── app.ini.example
│   │       └── public
│   │           └── assets
│   │               └── css
│   │                   └── nord_polar_night
│   └── forgejo.yml
├── jellyfin
│   ├── config
│   │   └── config
│   │       └── branding.xml
│   ├── jellyfin.yml
│   └── media
│       ├── movies
│       ├── music
│       └── shows
├── qbittorrent
│   ├── .env.example
│   ├── config
│   ├── downloads
│   ├── gluetun
│   └── qbittorrent.yml
```

## pre-install

### ignore lid closing

```
sudo mkdir -p /etc/systemd/logind.conf.d
sudo nano /etc/systemd/logind.conf.d/login.conf
```

```
[Login]
HandleLidSwitch=ignore
```

### hostname

```
sudo hostnamectl set-hostname fedora-server
```

### disable SElinux, for both jellyfin and qbittorrent work together - i will find better solution later
```
sudo grubby --update-kernel ALL --args="selinux=0"
```

```
sudo reboot
```

## install

```
sudo dnf install neovim git -y
```

```
cd ~
git clone https://github.com/hcg-leo/fedora-server
``` 

```
sudo chown -R $USER:$USER /home/hcg_leo/fedora-server
```

### [docker](https://github.com/docker)
```
sudo dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo
```

```
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

```
sudo systemctl enable --now docker
```

### [jellyfin](https://github.com/jellyfin/jellyfin)

```
sudo docker compose -f /home/hcg_leo/fedora-server/jellyfin/jellyfin.yml up -d
```

### [qbittorrent](https://github.com/qbittorrent/qBittorrent/) + [vpn](https://github.com/passteque/gluetun) - im using mullvad

```
cd /home/hcg_leo/fedora-server/qbittorrent
cp .env.example .env
nvim .env
```

```
sudo docker compose -f /home/hcg_leo/fedora-server/qbittorrent/qbittorrent.yml up -d
```

test at `https://ipleak.net/` - check if match with gluetun log

```
sudo docker logs gluetun
sudo docker logs qbittorrent
```

### [duckdns](https://github.com/linuxserver/docker-duckdns)

```
cd /home/hcg_leo/fedora-server/duckdns
cp .env.example .env
nvim .env
```

```
sudo docker compose -f /home/hcg_leo/fedora-server/duckdns/duckdns.yml up -d
```

### [forgejo](https://codeberg.org/forgejo)

```
cd /home/hcg_leo/fedora-server/forgejo/forgejo/gitea/conf
cp app.ini.example app.ini
```

```
sudo docker compose -f /home/hcg_leo/fedora-server/forgejo/forgejo.yml up -d
```

#### managing forgejo accounts

```
sudo docker exec -u git forgejo forgejo admin user create --admin --username hcg_leo --password 'password' --email aran20111118 [at] gmail [dot] com
```

then open 'http://hcg-leo.duckdns.org:3000/user/settings'
