# My VM setup

Install QEMU and Virt-Manager

<https://www.tecmint.com/install-qemu-kvm-ubuntu-create-virtual-machines/>

For Windows guest VM, continue here:

<https://sysguides.com/install-a-windows-11-virtual-machine-on-kvm/>

Download virtio drivers iso and mount it to virtual CD drive:

<https://sysguides.com/install-kvm-on-linux/#3-04-install-virtio-drivers-for-windows-guests->

Did not manage to get multi-monitor to work. Got it to work partially by editting XMl of Video VirtIO device under 'Show Virtual Hardware Details" to from 1 head to 2 or 3 head:

```xml
<video>
  <model type="virtio" heads="1" primary="yes">
    <acceleration accel3d="no"/>
  </model>
  <address type="pci" domain="0x0000" bus="0x00" slot="0x01" function="0x0"/>
</video>
```

Then switch to ```virt-viewer``` to view the VM while already running on Virt Manager.

Works partially because mouse and keyboard passthrough doesn't work when in ```virt-viewer```.

To get copy-paste to work between host and guest VM, install ```spice-guest-tools``` on guest VM, which includes spice-vd-agent.

If host is on KDE Plasma, Superkey will interfere with Windows key operation in Windows VM.  
To fix, while at the guest VM window, hit ```alt + F3``` (alternatively, right click the on title bar of the VM window), then click ```more actions``` -> ```Configure Special Window Settings``` -> ```Add property``` -> ```Appearance and Fixes``` -> ```Ignore global Shortcuts``` -> ```Force``` -> ```Yes``` -> ```Apply```.

More info on superkey issue: <https://www.reddit.com/r/kde/comments/7ueia5/when_using_virtualbox_vm_guests_the_superwindows/>

Using Remmina to connect remotely to the VMs work great! Connect to Windows VM using DRP solves all problems.

<https://www.digitalcitizen.life/connecting-windows-remote-desktop-ubuntu/>  

<https://www.digitalcitizen.life/enable-remote-desktop-windows/>  

To enable audio on Remmina, right-click on VM in list, `Edit` -> `Advanced` -> `Audio Output Mode` -> `Local`.

For MacOS VMs:

<https://christitus.com/macos-on-linux/>  

<https://computingforgeeks.com/how-to-run-macos-on-kvm-qemu/>  

<https://github.com/foxlet/macOS-Simple-KVM?tab=readme-ov-file>  

<https://github.com/foxlet/macOS-Simple-KVM/issues/168>  

<https://www.tecmint.com/connect-ubuntu-desktop-from-mac/#:~:text=the%20VNC%20client.-,Access%20Mac%20Remotely%20from%20Ubuntu,of%20your%20remote%20Mac%20desktop.&text=Next%2C%20provide%20the%20user%20credentials,press%20the%20%E2%80%9COK%E2%80%9D%20button.&text=Congrats>!

To fix screen resolution: <https://github.com/foxlet/macOS-Simple-KVM/blob/master/docs/guide-screen-resolution.md>

The `diskYsZ` referred is something that looks like `disk1s0`.

The point to press `Escape` is when the TianaCore screen appears. Pressing `Esc` here enters the 'bios' of the VM.

For Ubuntu VM:

Install extensions manager properly so that can instal dash to panel: <https://askubuntu.com/questions/1466647/extension-manager-crashes-on-ubuntu-23-04>

Need to use RDP beause VNC does not enable remote login, and gnome does not allow remote login: <https://askubuntu.com/questions/1411504/connect-when-remote-desktop-is-on-login-screen-or-screen-locked-without-autolog>

Somehow need to use proprietary remote system on Gnome for VNC: <https://askubuntu.com/questions/1461225/remmina-vnc-how-to-automatically-accept-connection>

Apparently gnome built-in RDP kinda works? <https://askubuntu.com/questions/1422263/rdp-on-ubuntu-desktop-22-04/1427768#1427768>

Should just setup xRDP using easy script: <https://c-nergy.be/blog/?cat=79>

Strategies to remotely connect to VM via internet: <https://www.helpwire.app/blog/remote-desktop-over-internet/>

Port forwarding for remote connection to VM: <https://askubuntu.com/questions/1267872/remote-desktop-access-between-2-ubuntu-20-04-devices-over-the-internet>

Make custom qcow2 image at easy accessible location (to be transferred elsewhere if so desired): <https://serverfault.com/questions/731417/how-do-you-create-a-qcow2-file-that-is-small-yet-commodious-on-a-linux-server>

```bash
qemu-img create -f qcow2 foobar.qcow2 100M
```

To resize qcow2 image: <https://gist.github.com/joseluisq/2fcf26ff1b9c59fe998b4fbfcc388342>

### Proxmox

On adding second drives to the computer to be added to be recognised by Proxmox: <https://forum.proxmox.com/threads/how-to-mount-second-partition-or-disk.13215/>

Proxmos post-install scripts (auto set no-sub repo): <https://tteck.github.io/Proxmox/>  
Also get dark theme from same link.

Resize local-lvm: <https://forum.proxmox.com/threads/resize-local-or-local-lvm-and-how.105263/>  
<https://youtu.be/_u8qTN3cCnQ?si=Sd3orctmIfyvu4a6&t=885>  

Set static local IP address on Ubuntu VMs: <https://www.freecodecamp.org/news/setting-a-static-ip-in-ubuntu-linux-ip-address-tutorial/>

Use SPICE to connect remotely to VMs on Proxmox: <https://www.reddit.com/r/homelab/comments/16e5fum/whats_your_preferred_remote_desktop_solution_to/>  

<https://codecisions.com/adding-spice-to-your-proxmox-virtual-machine/#:~:text=To%20connect%2C%20you%20need%20a,connect%20you%20to%20the%20VM>.

Can also self host a Guacamole instance for browser-based remote connection.

Virtual GPU passthrough: <https://www.youtube.com/watch?v=gkzrAYQHtk4>  
<https://forum.proxmox.com/threads/virtgl-proxmox-7-3-guides-on-how-to-set-up-what-kind-of-performance-to-expect.118575/>
<https://pve.proxmox.com/pve-docs/chapter-qm.html#qm_display>  
<https://forum.proxmox.com/threads/virglrenderer-for-3d-support.61801/page-3#post-466767>
VirGL drivers: <https://www.reddit.com/r/Proxmox/comments/v6p0om/amd_5750g_virgl_initial_benchmarks/>

Getting audio to work:
<https://forum.proxmox.com/threads/noob-guide-to-getting-audio-to-work-in-linux-guests.127369/>

Use `tasksel` to install Desktop Environment (applicable for all Linux distro).

Proper iGPU passthrough for AMD Ryzen 7 7735HS:
<https://pve.proxmox.com/wiki/PCI_Passthrough#AMD_specific_issues>  
reset bug: <https://www.nicksherlock.com/2020/11/working-around-the-amd-gpu-reset-bug-on-proxmox/#comment-60032>  
<https://forum.proxmox.com/threads/amd-rx-550-gpu-passthrough-issues.128405/#post-574818>  
Alternative instructions:
<https://forum.proxmox.com/threads/have-anyone-susscesfully-passthroughed-the-igpu-amd-radeon-680m-to-vm.119178/page-4>  
<https://github.com/isc30/ryzen-7000-series-proxmox>  

Forwarding port from VM inside proxmox to be accessed by network: <https://www.youtube.com/watch?v=RW1pwM94rvQ>

### Proxying

2 options:

1. Nginx Proxy Manager:
<https://www.youtube.com/watch?v=GarMdDTAZJo>

2. Cloudflare Tunnel
<https://noted.lol/say-goodbye-to-reverse-proxy-and-hello-to-cloudflare-tunnels/>
<https://www.youtube.com/watch?v=hrwoKO7LMzk>

Can also look into Traefik

### Remote access

Use PiVPN <https://www.pivpn.io/>, set up wireguard.

XPipe for GUI app for making and managing connections (file manager, docker, etc)
<https://www.youtube.com/watch?v=wjd3E0EN2xk>

### Debian shenanigans

Don't bother with Debian. Kubuntu works perfectly with audio working and xRDP install no issues. Was not able to get xRDP working on Debian.

Not in sudoers' file: <https://www.youtube.com/watch?v=okuO3fRbupw>
