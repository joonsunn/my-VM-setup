# Arch notes

## `archinstall` configs

Under `Network Configuration`, select "copy network configuration".

## UFW

```bash
pacman -Sy ufw
ufw allow ssh/tcp
systemctl start ufw
systemctl enable ufw
systemctl status ufw
ufw enable
```

## SSH

```bash
sudo pacman -S openssh
sudo systemctl start sshd
sudo systemctl enable sshd
sudo ufw allow 22/tcp
```

## Install `yay`

```bash
sudo pacman -Syu
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
