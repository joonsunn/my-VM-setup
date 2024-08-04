# Setting up KDE Neon 6.0

## Basic Setup

```bash
sudo apt update -y
sudo apt upgrade -y
sudo apt install qemu-guest-agent openssh-server chromium-browser -y
```

## Setup xRDP

```bash
wget https://www.c-nergy.be/downloads/xRDP/xrdp-installer-1.5.1.zip
unzip xrdp-installer-1.5.1.zip
chmod +x xrdp-installer-1.5.1.sh
```

Add to ~line 210 of `xrdp-installer-1.5.1.sh`:

```bash
nano xrdp-installer-1.5.1.sh
```

```bash
"plasmax11")
    DesktopVer="KDE"
    SessionVer=""
    /bin/echo -e "\e[1;32m       |-| Session         : $SessionVer\e[0m"
    /bin/echo -e "\e[1;32m       |-| Desktop Version : $DesktopVer\e[0m"
    ;;
```

Add to ~line 374:

```bash
   *"KDE neon 6.0"*)
   /bin/echo -e "\e[1;32m       |-| OS Version : $version\e[0m"
   /bin/echo -e "\e[1;32m       |-| Desktop Version : $DesktopVer\e[0m"
    ;;
```

```bash
./xrdp-installer-1.5.1.sh -s
```

```bash
sudo nano /etc/xrdp/xrdp.ini
```

Make following changes:

```conf
tcp_send_buffer_bytes=4194304
crypt_level=low
```

## Setup Docker

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

## Setup NodeJS

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install lts
nvm use lts
npm install -g npm@latest
```

## Install VSCode

```bash
sudo apt-get install wget gpg -y
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
rm -f packages.microsoft.gpg
sudo apt install apt-transport-https -y
sudo apt update
sudo apt install code -y
```

## Setup sudo

```bash
visudo
```

Scroll down until the line that says:

```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
```

Add `newuser` to below `root`:

```bash
newuser    ALL=(ALL:ALL) ALL
```

Save and exit file.

## Setup SSH for GitHub

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
exec ssh-agent bash
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

```bash
ssh -T git@github.com
```

```bash
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
git config --global core.editor "code --wait"
```

## Setup Plasma

Right click Panel -> `Show Panel Configuration` -> `Floating` -> Toggle off  
Right click Panel -> `Show Alternatives...` -> `Icons and Text Task Manager`  
Right click Panel -> `Configure Icons and Text Task Manager` -> `Behavior` -> `Group` -> `Do Not Group`

`System Settings`:  
`Colors and Themes` -> `Breeze dark`  
`Window Decoration` -> `Get New` -> `Utterly Round Dark`

Date format (Sun, 04 Aug 2024):

```text
ddd, dd MMM yyyy
```

## Setup Sunshine

Host/server side, installed the latest package at LizardByte github: <https://github.com/LizardByte/Sunshine/releases>

After downloading the `sunshine-ubuntu-22.04-{arch}.deb` file, install:

```bash
sudo apt install -f ./sunshine-{ubuntu-version}.deb
```

```bash
sunshine
```

Point browser to `https://localhost:47990` on host/server, then set up `username` and `password`.

Turn on `UPnP` in Sunshine Settings under `Network`.

For mouse cursor not appearing in Wayland:

Create `/etc/environment.d/50-kwin-sw-cursor.conf` with following content (to disable hardware cursor): <https://github.com/LizardByte/Sunshine/issues/93>

```props
KWIN_FORCE_SW_CURSOR=1
```

### Sunshine autostart

`autostart-sunshine.sh`:

```bash
#!/bin/bash
sleep 1
xdg-screensaver lock
```

Then under `System Settings` -> `Autostart` -> `+ Add...` -> `Add login script`, click `add` then select `autostart-sunshine.sh`.

Then set `Login screen` -> Automatically Log in as user: [user] with session [Plasma (Wayland)].

Add `sunshine` to `add autostart application`.

## Set custom resolution (e.g. 1440p)

<https://askubuntu.com/questions/377937/how-do-i-set-a-custom-resolution>

Create a file `~/.xprofile`:

```bash
xrandr --newmode "2560x1440_60.00"  312.25  2560 2752 3024 3488  1440 1443 1448 1493 -hsync +vsync
xrandr --addmode Virtual-1 2560x1440_60.00
```

If on Wayland: <https://davejansen.com/add-custom-resolution-and-refresh-rate-when-using-wayland-gnome/>

Check display name:

```bash
ls /sys/class/drm/card*
```

Or

```bash
xrandr --listactivemonitors
```

Then:

```bash
sudo nano /etc/default/grub
```

```diff
- GRUB_CMDLINE_LINUX_DEFAULT="quiet video"
+ GRUB_CMDLINE_LINUX_DEFAULT="quiet video=Virtual-1:2560x1440@60"
```

Example above uses 'Virtual-1' as display name.

Then run:

```bash
update-grub
```

## Chrome Extensions

- Ublock Origin <https://chromewebstore.google.com/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm?hl=en>
- SponsorBlock <https://chromewebstore.google.com/detail/sponsorblock-for-youtube/mnjggcdmjocbbbhaepdhchncahnbgone>
- Dearrow <https://chromewebstore.google.com/detail/dearrow-better-titles-and/enamippconapkdmgfgjchkhakpfinmaj>
- Enhancer For YouTube <https://chromewebstore.google.com/detail/enhancer-for-youtube/ponfpcnoihfmfllpaingbgckeeldkhle>
- Reddit Enhancement Suite <https://chromewebstore.google.com/detail/reddit-enhancement-suite/kbmfpngjjgdllneeigpgjifpgocmfgmb>
- Search By Image <https://chromewebstore.google.com/detail/search-by-image/cnojnbdhbhnkbcieeekonklommdnndci>
- JSON Viewer <https://chromewebstore.google.com/detail/json-viewer/gbmdgpbipfallnflgajpaliibnhdgobh>
- React Developer Tools <https://chromewebstore.google.com/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi>
