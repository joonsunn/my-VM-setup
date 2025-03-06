# Arch notes

## `archinstall` configs

Under `Network Configuration`, select "Use Network Manager".  
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

## Tailscale (required for Sunshine to work)

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

## Sunshine

```bash
yay sunshine
sudo setcap cap_sys_admin+p $(readlink -f $(which sunshine))
systemctl --user start sunshine
systemctl --user enable sunshine
```

Add the following to `/etc/environment`

```bash
KWIN_FORCE_SW_CURSOR=1
```

### Set up `sunshine` auto start

Basically sunshine can only autorun right after login. So need to set up auto login that immediately locks the screen.

In KDE Plasma, need to enable autologin:

`Settings` -> `Color & Themes` -> `Login Screen (SDDM)` -> `Behaviour` -> Set automatically login, uncheck login after logout.

Then create a `.sh` file anywhere with following content:

`autostart-sunshine.sh`

```bash
#!/bin/bash
sleep 1
xdg-screensaver lock
```

Then set as login script:
`Settings` -> `Autostart` -> `Add new` -> `Login script` -> Select the file created earlier.

If GDM is still running, need to disable it:

```bash
sudo systemctl disable gdm
```

