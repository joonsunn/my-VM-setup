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
