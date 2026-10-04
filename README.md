# fedora server config

_self host!!!_ - fedora server running jellyfin + qbittorrent, vpn binded to just qbittorrent through gluetun

```
.
├── .gitignore
├── README.md
├── jellyfin
│   ├── config
│   │   └── config
│   │       ├── branding.xml
│   │       └── system.xml
│   ├── jellyfin.yml
│   └── media
│       ├── movies
│       └── shows
└── qbittorrent
    ├── .env.example
    ├── config
    ├── downloads
    ├── gluetun
    └── qbittorrent.yml
```

### install

```
sudo dnf install neovim -y
```

```
sudo dnf install git -y
```

ignore lid closing thing:

```
sudo mkdir -p /etc/systemd/logind.conf.d
sudo nvim /etc/systemd/logind.conf.d/login.conf
```

```
[Login]
HandleLidSwitch=ignore
```

```
cd ~
git clone https://github.com/hcg-leo/fedora-server
```

### jellyfin

```
sudo docker compose -f jellyfin.yml up -d
```

### qbittorrent + vpn - im using mullvad

```
cd /home/hcg_leo/fedora-server/qbittorrent/gluetun
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

### need to test if this works or if it was disabling SElinux

```
sudo chmod -R 777 /home/hcg_leo/fedora-server/
```
