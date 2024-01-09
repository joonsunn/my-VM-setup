# My Proxmox Experience log

## Setup mini PC (SZBOX 7735HS)

Relatively straightforward installation of RAM and SSD. Top cover tricky to remove but all is well. Front USB port is extremely tight. Needed to use back USB port to properly plug in USB drive.

Unable to use Ventoy to install. Ventoy boots up fine but Proxmox does not like it. Made standalone Proxmox USB boot drive (using Etcher) and it worked well.

During installation, used following settings for network:

```bash
Network management interface: eno1
hostname: pve.local
IP Address: XXX.XXX.XXX.10/24 (XXX == first 3 groups of number in default gateway IP address)
Gateway: XXX.XXX.XXX.XXX (exactly as per default gateway, check router/mesh settings)
DNS Server: 8.8.8.8
```

## Networking

Current home network is:

Internet --> Modem Router --> Mesh --> Devices

TP Link Deco's Guest wifi network is isolated from the rest of the network. Need to use non-guest network to see local devices. (should have tested with ping to other local devices to check for local network IP isolation, if any)

Also unable to ping local devices if remote computer is on VPN (e.g. NordVPN).

EDIT: adding subnet to whitelist worked: <https://askubuntu.com/questions/1276829/nordvpn-local-network-addresses-not-reachable-when-connected>

```bash
nordvpn whitelist add subnet 192.168.[subnet].0/24
```

After set up as above, finally can connect to Proxmox web UI. However unable to ping google.com from shell. Checked network settings to be as follows (autostart and VLAN aware):

![Alt text](image.png)

Also added secondary DNS `1.1.1.1`.

After this, ping Google worked.

## Post Install Scripts

Installed "Proxmox VE post install" script from here: <https://tteck.github.io/Proxmox/>

This script was run with all "yes". Took a while to complete because Proxmox was also updated in the process. (ran this script right after I got network working)

Did not need to install dark mode script as Proxmox 8.1 came default with dark theme enabled and activated.

Small scare as after post-install script completed, was unable to connect to it after reboot even after checking for wifi and VPN and ping to the IP was ok. But needed to wait an extra bit while before it all came back online.

## Expanding available space

Based on this video: <https://www.youtube.com/watch?v=_u8qTN3cCnQ&t=885s>

At Datacenter -> Storage -> `local-lvm` -> Remove.  
At PVE node -> Shell ->  

```bash
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```

After that, ```local``` still will not allow VM to be installed. `local` need to be explicitly allowed to install disk images to install VMs. To do this:

At Datacenter -> Storage -> `local` -> Edit -> Content -> Click everything.

VM creation should now be possible in `local`.

## VirGL on Proxmox VE

Ran the following commands:

```bash
sudo apt-get install libgl1-mesa-dev mesa-common-dev mesa-utils
sudo apt install libgl1 libegl1
```

2nd command above returned that all libraries are latest, so it could be unnecessary to run.

## Kubuntu VM creation

First creation: all defaults, 50GB, 16GB RAM, 8 cores. Tick Qemu-Agent checkbox.
Bunch of SQUASHFS errors appeared during first reboot after installation. Does not want to reboot properly. Killed it with force off, and it booted up normally.

```bash
sudo apt update
sudo apt upgrade
```

Install QEMU-agent to get IP-address to show up in Proxmox VM summary page: <https://pve.proxmox.com/wiki/Qemu-guest-agent>

```bash
sudo apt-get install qemu-guest-agent
```

At this point, sound does not work on the VM through web ui shell connection session (volume icon showing disabled/not available).

Install XRDP Easy Install script (just google and navigate the website to get latest script URI): <https://c-nergy.be/blog/?cat=79>

```bash
wget https://www.c-nergy.be/downloads/xRDP/xrdp-installer-1.4.8.zip
unzip xrdp-installer-1.4.8.zip 
chmod +x  xrdp-installer-1.4.8.sh
./xrdp-installer-1.4.8.sh -s
```

Remember the `-s` flag!!!
TODO: Investigate the `-c` flag for custom installation with compilation from source. Necessary?

Machine settings: <https://forum.proxmox.com/threads/solved-vms-linux-and-windows-very-slow-and-laggy.104469/>  
Changed machine to q35  
Changed CPU to HOST  

At destination machine:

```bash
sudo nano /etc/xrdp/xrdp.ini
```

Make following changes:

```bash
tcp_send_buffer_bytes=4194304
crypt_level=low
```

Remmina settings:

Color settings: GFX 32bpp  
Network: LAN  

![Alt text](image-1.png)

Seem like adding `latency:175` made an actual difference to sounds latency, less crackly/choppy sound.

![Alt text](image-5.png)

From official docs:
![Alt text](image-6.png)

Fix choppy audio by making sure `pulseaudio` is started:

```bash
pulseaudio --version
pulseaudio --start
```

Can try restart `pulseaudio`

```bash
pulseaudio -k
```

Somehow sometimes running `-k` after starting `pulseaudio` makes audio smoother. Not sure why.

If `pulseaudio` not installed: <https://itslinuxfoss.com/install-pulseaudio-ubuntu-22-04/>

Also tried uninstall default Remmina, and install the Flatpak version. On KDE Plasma need to install the Flatpak backend support plugin first, but the Remmina Flatpak file will automatically prompt to instll it on first click. <https://flathub.org/apps/org.remmina.Remmina>

For some reason, Flatpak Remmina is light theme only.

Flatpak Remmina has more codec options

![Alt text](image-4.png)

But Flatpak version uses different directory for settings, so previous changes made in Snap Remmina is not carried over.

Alternative remote desktop solution: FreeRDP

check if FreeRDP is installed:

```bash
xfreerdp
```

Then make connection:

```bash
xfreerdp -f /u:[username] /p:[password] /v:[IP of destination] /sound:sys:pulse /dynamic-resolution +clipboard /network:lan /gfx:rfx /bpp:32 /sound:latency:20
```

Not sure how to get fullscreen toggle to work, because `CRTL+ALT+ENTER` never worked for me. Due to this, I still prefer Remmina. But need to find a way to make the options bar in Remmina dark instead of current white theme.

## Misc

To check wifi from terminal: <https://askubuntu.com/questions/1401941/is-there-a-way-to-check-whether-my-device-is-currently-using-the-2-4-ghz-or-5-gh>

```bash
nmcli device wifi list
```

To connect to specific BSSID:  
`Settings` -> `Connections` -> `BBSID` dropdown

![Alt text](image-2.png)

Use XPIPE as a general SSH manager: <https://xpipe.io/download>
Mainly use XPIPE to send files.

Need to start `ssh` server on host PC:

```bash
sudo apt-get install openssh-server
sudo service ssh status
```

Then need to allow port 22 in `ufw`:

```bash
sudo ufw allow ssh 
```

For RDP apparently proper installation requires opening port 3389 on UFW:

```bash
sudo ufw allow 3389
```

Then:

```bash
sudo ufw status
```

If not started, then

```bash
sudo ufw enable
```

To kill firewall:

```bash
sudo ufw disable
```

## virGL on Ubuntu Vm

Install VirGL drivers: <https://www.reddit.com/r/Proxmox/comments/v6p0om/amd_5750g_virgl_initial_benchmarks/>  

Only able to get `glxgears -info` to show virGL when accessing VM through Proxmox shell. When accessing through xRDP, can only get `llvmpipe` no matter what.  
  
Other ways to check:  

`glxinfo -B`  
`dmesg | grep vga`  
`dmesg | grep virgl`  

Went through GPU passthrough, but still showing `llvmpipe`, with suboptimal FPS when playing youtube 1080p60fps.  

Even went as far as modifying xRDP Easy Install script to include `--enable glamour` at the vonfigure step of `XRDP` and `XORG`, still no changes. Info from : <https://gist.github.com/rkkoszewski/aa531cee7126edf329b76bdd0546f502>  

TODO: To investigate further.  

Perhaps `X11Forwarding` is the answer? <https://stackoverflow.com/questions/61590691/the-xauthority-file-is-not-does-not-existhence-via-local-ssh-connection-displa>


On switching display manager: <https://techpiezo.com/linux/switch-display-manager-in-ubuntu-20-04/>  

On `lightdm` greeter: <https://www.reddit.com/r/archlinux/comments/nr6sb2/new_to_arch_cant_start_lightdm/>  
 
## Windows VM creation

Need to pre-load virt-io driver iso, because drivers are needed to detect drives in the first step of installation. Select the appropriate `win` version when loading drivers to proceed.

Initial hiccup is that Windows 11 installation unable to proceed without network connection (no button to click next), need to reboot into `no network mode`: <https://learn.microsoft.com/en-us/answers/questions/1179311/windows-11-setup-without-internet>

Click `SHIFT+F10`

In the command prompt that appears:

`OOBE\BYPASSNRO`

Installation will reboot and proceed. However upon first boot, network will be unavailable. So reboot and network will be working.

First thing to install is virt-io drivers and virtio-win-guest-tools from the virtio iso.

Then need to install ExplorePatcher to make start menu show all apps instead of pinned apps: <https://github.com/valinet/ExplorerPatcher>

For RDP: Enable RDP in RDP settings.

Sound works out of the box in Win11. Youtube on Chrome works great without additional settings.

For max scrolling performance, use `GFX AVC444` color depth in Remmina.

Display scaling is wonky on Win11 VM via RDP. The clock at the taskbar is never affected by any DPI or text size setting. Just set Display Scaling to 125% and be done with it.

VirGL drivers for Windows still work in progress: <https://github.com/virtio-win/kvm-guest-drivers-windows/pull/943>
