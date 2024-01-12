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
chmod +x xrdp-installer-1.4.8.sh
nano xrdp-installer-1.4.8.sh
===== add --enable-glamor flags in configure. See below =====
 ./xrdp-installer-1.4.8.sh -s
```

Modify xRDP Easy Install script to include `--enable glamour` at the vonfigure step of `XRDP` and `XORG`, still no changes to virGL over XRDP. Info from : <https://gist.github.com/rkkoszewski/aa531cee7126edf329b76bdd0546f502>  

`--enable-glamor --enable-rfxcodec --enable-mp3lame --enable-fdkaac --enable-opus --enable-pixman --enable-fuse --enable-jpeg --enable-ipv6`

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

TODO: To investigate further.  

Check Virtualization is available on CPU:

```bash
cat /proc/cpuinfo | grep vmx svm
```

Perhaps `X11Forwarding` is the answer? <https://stackoverflow.com/questions/61590691/the-xauthority-file-is-not-does-not-existhence-via-local-ssh-connection-displa>

On switching display manager: <https://techpiezo.com/linux/switch-display-manager-in-ubuntu-20-04/>  

On `lightdm` greeter: <https://www.reddit.com/r/archlinux/comments/nr6sb2/new_to_arch_cant_start_lightdm/>  

Forum thread to follow on progress of virGL (virtio-vga-gl): <https://forum.proxmox.com/threads/virglrenderer-for-3d-support.61801/page-4#:~:text=Yes%2C%20VGA%20==%20GPU%2C%20so%20VirtioGPU%20is,CLI/API%20or%20even%20Web%20UI.%20chrcoluk%20said>:.

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

## GPU Passthrough

Below procedure taken from: <https://forum.proxmox.com/threads/pci-gpu-passthrough-on-proxmox-ve-8-installation-and-configuration.130218/>

1) editting `/etc/default/grub`

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet  iommu=pt"
update-grub
reboot
```

Alternative config if passthrough fails:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt nomodeset pcie_acs_override=downstream initcall_blacklist=sysfb_init"
```

Another alternative config:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt pcie_acs_override=downstream,multifunction nofb nomodeset video=vesafb:off,efifb:off"
```

Extreme config:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt pcie_acs_override=downstream,multifunction video=efifb:off video=vesa:off vfio-pci.ids=10de:13bb,10de:0fb vfio_iommu_type1.allow_unsafe_interrupts=1 kvm.ignore_msrs=1 modprobe.blacklist=radeon,nouveau,nvidia,nvidiafb,nvidia-gpu"
```

From: <https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/>  
Also: <https://www.reddit.com/r/Proxmox/comments/lcnn5w/proxmox_pcie_passthrough_in_2_minutes/>  
Further reading: <https://www.reddit.com/r/homelab/comments/11l0s5j/boxmox_asrock_4x4_box_5800u_jbod_proxmox/>  
Graphical gudie (ignore part about vfio_virqfd): <https://3os.org/infrastructure/proxmox/gpu-passthrough/igpu-passthrough-to-vm/#linux-virtual-machine-igpu-passthrough-configuration>  
More: <https://github.com/isc30/ryzen-7000-series-proxmox>  

2) editting `/etc/modules`

Add the following lines:

```bash
vfio
vfio_iommu_type1
vfio_pci
```

Then run:

```bash
update-initramfs -u -k all
systemctl reboot
```

Verify status of VFIO:

```bash
dmesg | grep -i vfio
```

Should see something similar to:`[ 7.262027] VFIO - User Level meta-driver version: 0.3`

3) Verify remapping:

```bash
dmesg | grep 'remapping'
```

Should see `AMD-Vi: Interrupt remapping enabled` or `DMAR-IR: Enabled IRQ remapping in x2apic mode`

Else:

```bash
echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
```

4) Additional AMD setup (and driver block):

```bash
apt install pve-headers-$(uname -r)
apt install git dkms build-essential
git clone https://github.com/gnif/vendor-reset.git
cd vendor-reset
dkms install .
echo "vendor-reset" >> /etc/modules
update-initramfs -u
shutdown -r now
```

Retrieve PCI ID of GPU:

```bash
lspci -nn | grep 'AMD'
```

For 7735HS, results include:

```bash
...
05:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Rembrandt [Radeon 680M] [1002:1681] (rev 0a)
05:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Rembrandt Radeon High Definition Audio Controller [1002:1640]
...
```

In this case, PCI ID for GPU is `05:00.0`.

Full extracted info is:

```bash
GPU: 1002:1681
Audio: 1002:1640
```

Next create the service for vendor reset (AMD bug):

```bash
cat << EOF >>  /etc/systemd/system/vreset.service
[Unit]
Description=AMD GPU reset method to 'device_specific'
After=multi-user.target
[Service]
ExecStart=/usr/bin/bash -c 'echo device_specific > /sys/bus/pci/devices/0000:05:00.0/reset_method'
[Install]
WantedBy=multi-user.target
EOF
systemctl enable vreset.service && systemctl start vreset.service
```

Should see a bunch of dmesg during start up (example, 7735HS is not POLARIS):

```bash
[57709.971750] vfio-pci 0000:01:00.0: AMD_POLARIS10: version 1.1
[57709.971755] vfio-pci 0000:01:00.0: AMD_POLARIS10: performing pre-reset
[57709.971881] vfio-pci 0000:01:00.0: AMD_POLARIS10: performing reset
[57709.971885] vfio-pci 0000:01:00.0: AMD_POLARIS10: CLOCK_CNTL: 0x0, PC: 0x2055c
[57709.971889] vfio-pci 0000:01:00.0: AMD_POLARIS10: Performing BACO reset
[57710.147491] vfio-pci 0000:01:00.0: AMD_POLARIS10: performing post-reset
[57710.171814] vfio-pci 0000:01:00.0: AMD_POLARIS10: reset result = 0
```

Perform Soft deps to block drivers from loading on host:

```bash
echo "options vfio-pci ids=1002:1681,1002:1640" >> /etc/modprobe.d/vfio.conf
# For AMD
echo "softdep radeon pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
echo "softdep amdgpu pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
echo "softdep snd_hda_intel pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
echo "softdep xhci_pci pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
# For Nvidia
echo "softdep nouveau pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
echo "softdep nvidia pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
echo "softdep nvidiafb pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
echo "softdep nvidia_drm pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
echo "softdep drm pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
# For Intel
echo "softdep snd_hda_intel pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
echo "softdep snd_hda_codec_hdmi pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
echo "softdep i915 pre: vfio-pci" >> /etc/modprobe.d/vfio.conf
```

If softdeps don't work, then need to proceed to blacklist drivers:

```bash
echo "options vfio-pci ids=1002:1681,1002:1640" >> /etc/modprobe.d/vfio.conf
```

Skip above if already done before.

```bash
# AMD drivers
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
echo "blacklist amdgpu" >> /etc/modprobe.d/blacklist.conf
# NVIDIA drivers
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidiafb" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia_drm" >> /etc/modprobe.d/blacklist.conf
# Intel drivers
echo "snd_hda_intel" >> /etc/modprobe.d/blacklist.conf
echo "snd_hda_codec_hdmi" >> /etc/modprobe.d/blacklist.conf
echo "i915" >> /etc/modprobe.d/blacklist.conf
```

Additional checking stuff can be found from Proxmox forum post above.

In the VM conf file: <https://nopresearcher.github.io/Proxmox-GPU-Passthrough-Ubuntu/>

```bash
nano /etc/pve/qemu-server/(VMID).conf
hostpci0: 01:00,x-vga=on
```

Make sure `,x-vga=on` is included.

## Moonlight+Sunshine

Having xRDP stubbornly only use `llvmpipe`, and GPU passthrough seeming to not make a difference over xRDP, decided to use Moonlight+Sunshine.

Host/server side, installed the latest package at LizardByte github: <https://github.com/LizardByte/Sunshine/releases>

After downloading the `sunshine-ubuntu-22.04-{arch}.deb` file, install:

```bash
sudo apt install -f ./sunshine-{ubuntu-version}.deb
```

Before starting `sunshine` service, need to login to VM from Proxmox shell, just in case previous step was done via xRDP or other means. Starting `sunshine` will not work over xRDP because it needs a display to attach to, which xRDP is alreayd occupying if currenty connected via xRDP.

To start `sunshine` service after installed and logged in via Proxmox shell, run:

```bash
sunshine
```

Then a bunch of mumbo jumpa will appear in the terminal, culminating with something that looks like:

```bash
Configuration UI available at [https://localhost:47990]
```

Point browser to `https://localhost:47990` on host/server, then set up `username` and `password`.

Then install `moonlight` on client PC: <https://github.com/moonlight-stream/moonlight-qt/releases>

I chose to install via Snap store: <https://snapcraft.io/moonlight> (previous attempt using Flatpak yielded subpar results.)

```bash
sudo snap install moonlight
```

Then open `moonlight` from app menu. Had to manually add host/server IP via [+] icon at top right to get it to detect.

Moonlight settings:

```bash
resolution: 1920x1080
frame rate: 60fps
Video bitrate: 20Mbps (higher causes crackly audio)
Optimise mouse for remote desktop: OFF
```

Need to have access to both client and server to establish connection, because need to type in PIN at server side to pair the devices for connection.

Full desktop is piped through. Since no new session is started, no problem with virGL.

If got problem establishing connection, might have to open ports on server side. Required ports are shown in Web UI -> Advanced.

Downside to `moonlight` is that need to login to server device to start the `sunshine` service, which means no autologin/start on bootup.
