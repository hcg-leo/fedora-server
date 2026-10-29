# fedora server config

self hosting some stuff, all custom, and all [nord theme](https://github.com/nordtheme/nord)

```
.
├── caddy
│   ├── caddyfile
│   └── caddy.yml
├── couchdb
│   ├── couchdb.yml.example
│   ├── data
│   └── etc
├── duckdns
│   └── duckdns.yml
├── forgejo
│   ├── forgejo
│   │   └── gitea
│   │       ├── conf
│   │       │   └── app.ini.example
│   │       └── public
│   │           └── assets
│   │               └── css
│   │                   └── theme-nord.css
│   └── forgejo.yml
├── .gitignore
├── jellyfin
│   ├── config
│   │   └── config
│   │       └── branding.xml
│   ├── jellyfin.yml
│   └── media
│       ├── movies
│       ├── music
│       └── shows
├── kavita
│   ├── books
│   ├── config
│   │   └── themes
│   │       └── nord.css
│   ├── kavita.yml
│   └── manga
├── qbittorrent
│   ├── config
│   ├── downloads
│   ├── gluetun
│   └── qbittorrent.yml
└── README.md
```
> `git ls-files | tree --fromfile .` to make the cool tree

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

### docker
```
sudo dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo
```

```
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

```
sudo systemctl enable --now docker
```

### duckdns

```
cd /home/hcg_leo/fedora-server/duckdns
cp .env.example .env
nvim .env
```

```
sudo docker compose -f /home/hcg_leo/fedora-server/duckdns/duckdns.yml up -d
```

```
http://192.168.1.254/
```

### caddy
```
sudo docker compose -f /home/hcg_leo/fedora-server/caddy/caddy.yml up -d
```

### jellyfin

```
sudo docker compose -f /home/hcg_leo/fedora-server/jellyfin/jellyfin.yml up -d
```

> configure user spaces yourself sorry 

### qbittorrent

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
sudo docker logs qbittorrent
```

### forgejo

```
cd /home/hcg_leo/fedora-server/forgejo/forgejo/gitea/conf
cp app.ini.example app.ini
```

```
sudo docker compose -f /home/hcg_leo/fedora-server/forgejo/forgejo.yml up -d
```

#### managing forgejo accounts

```
sudo docker exec -u git forgejo forgejo admin user create --admin --username hcg-leo --password 'password' --email aran20111118@gmail.com
```

then open `https://forgejo.hcg-leo.duckdns.org/user/settings`

### couchdb for obsidian live sync

```
cd /home/hcg_leo/fedora-server/couchdb
cp couchdb.yml.example couchdb.yml
nvim couchdb.yml
```

```
sudo docker compose -f /home/hcg_leo/fedora-server/couchdb/couchdb.yml up -d
```

### kavita

```
sudo docker compose -f /home/hcg_leo/fedora-server/kavita/kavita.yml up -d
```
