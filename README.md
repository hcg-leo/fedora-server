sudo dnf install neovim -y

sudo mkdir -p /etc/systemd/logind.conf.d
sudo nvim /etc/systemd/logind.conf.d/login.conf
[Login]
HandleLidSwitch=ignore

sudo dnf install git -y

sudo docker compose -f jellyfin.yml up -d

https://github.com/IAmParadox27/jellyfin-plugin-media-bar

cd /home/hcg_leo/fedora-server/qbittorrent/gluetun
cp .env.example .env
nvim .env

sudo docker compose -f /home/hcg_leo/fedora-server/qbittorrent/qbittorrent.yml up -d
