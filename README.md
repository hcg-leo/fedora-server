# fedora server config

self hosting some stuff, all custom, and all [nord theme](https://github.com/nordtheme/nord)

```
.
├── caddy
│   ├── caddyfile
│   ├── caddy.yml
│   ├── config
│   └── data
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
│       └── user
│           ├── movies
│           ├── music
│           └── shows
├── qbittorrent
│   ├── config
│   ├── downloads
│   ├── gluetun
│   └── qbittorrent.yml
├── README.md
├── vikunja
│   ├── config.yml
│   ├── db
│   ├── files
│   └── vikunja.yml
└── webdav
    ├── data
    └── webdav.yml
```

> `git rm -r --cached .` `git add .` and `git ls-files | tree --fromfile .` to make the cool tree

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

get the wireguard key from `https://account.protonvpn.com/downloads` wireguard configuration. remember to turn on `NAT-PMP`. copy the `PrivateKey` into `.env`

```
cd /home/hcg_leo/fedora-server/qbittorrent
cp .env.example .env
nvim .env
```

```
sudo docker compose -f /home/hcg_leo/fedora-server/qbittorrent/qbittorrent.yml up -d
```

get the temporary web ui password

```
sudo docker logs qbittorrent
```

open qBittorrents web ui, go to settings --> web ui tab --> auth and tick `bypass authentication for clients on localhost`

```
sudo docker compose -f /home/hcg_leo/fedora-server/qbittorrent/qbittorrent.yml up -d --force-recreate
```

test at `https://ipleak.net/` - check if match with gluetun log

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
sudo docker exec -u git forgejo forgejo admin user create --admin --username hcg-leo --password 'password' --email 'email'
```

then go to `https://forgejo.hcg-leo.duckdns.org/user/settings`

#### open with option for zed editor

go to https://forgejo.hcg-leo.duckdns.org/admin/config/settings and add the thing bellow

```
zed = zed://git/clone?repo={url}
```

### webdav for obsidian sync

```
cd /home/hcg_leo/fedora-server/webdav
cp .env.example .env
nvim .env
```

```
sudo docker compose -f /home/hcg_leo/fedora-server/webdav/webdav.yml up -d
```

install the `remotely save` community plugin and set:

- remote service: `WebDAV` - couchdb too complicated for this
- server address: `https://webdav.hcg-leo.duckdns.org`
- username / password: same as `.env`
- auth type: `basic`
- run on startup: `1 second`
- save on sync: `enabled`

### vikunja

```
cd /home/hcg_leo/fedora-server/vikunja
echo "SECRET=$(od -An -tx1 -N32 /dev/urandom | tr -d ' \n')" > .env
```

```
sudo docker compose -f /home/hcg_leo/fedora-server/vikunja/vikunja.yml up -d
sudo docker restart caddy
```

```
sudo docker exec -it vikunja /app/vikunja/vikunja user create -u hcg-leo -e 'email'
```
