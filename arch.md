# Arch notes

## `archinstall` configs

Under `Network Configuration`, select "copy network configuration".
Under `Audio`, select `pipewire`

## `qemu-guest-agent`
```bash
pacman -S qemu-guest-agent
```

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

## Install `xrdp`

Choose `1 5 14` after `yay xorg` to select the following:

```bash
14 aur/pipewire-module-xrdp 0.2-1 (+0 0.00)
    xrdp pipewire module
5 aur/xorgxrdp 0.10.3-1 (+17 0.30)
    Xorg drivers for xrdp
1 aur/xrdp 0.10.2-4 (+148 1.07)
    An open source remote desktop protocol (RDP) server
```
    
```bash
yay xorg
yay xorgxrdp
yay pipewire-module-xrdp
1 5 14
```

Create two files:

`/etc/X11/Xwrapper.config`

```bash
allowed_users=anybody
needs_root_rights=no
```

`~/.xinitrc`

```bash
/usr/lib/plasma-dbus-run-session-if-needed startplasma-x11
```

Then enable xrdp in systemctl

```bash
sudo systemctl enable xrdp
sudo systemctl start xrdp
sudo ufw allow 3389
```
