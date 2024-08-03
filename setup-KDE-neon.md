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
