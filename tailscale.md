# setup Tailscale + NordVPN

Run Tailscale from LXC: <https://www.youtube.com/watch?v=QJzjJozAYJo>

On initial setup of LXC, set IPv4 as DHCP first, and IPv6 static.

Then enable ssh password:

```bash
nano /etc/ssh/sshd_config
```

Change to the following lines:

```properties
PermitRootLogin yes
```

Then run `ip a | grep 192`, and note the `192` ip address.

Change LXC's network settings -> IPv4 -> static

```txt
IPv4/CIDR: 192.168.68.120/24
Gateway: 192.168.68.1 (default gateway)
```

Then

```bash
ping 1.1.1.1
```

Might need to run as root to execute above command.

```bash
apt update && apt upgrade -y
```

Then install tailscale (check latest tailscale docs):

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Do not login to Tailscale yet.

```bash
nano /etc/sysctl.conf
```

Make the following changes:

```properties
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

Then kill the container. In PVE console:

```bash
nano /etc/pve/lxc/112.conf (replace with LXC id number)
```

Then paste the following two lines (https://tailscale.com/kb/1130/lxc-unprivileged):

```properties
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

Run the LXC.

Then run

```bash
tailscale up --advertise-routes=192.168.68.0/24 --advertise-exit-node
```

Then login accordingly. Approve the pending subnet and exit node requests.

Then enable UFW for tailscale0

```bash
sudo ufw allow in on tailscale0
sudo ufw allow out on tailscale0
sudo ufw allow ssh
sudo ufw enable
```

also

```bash
sudo systemctl enable tailscaled
```

Then at client side:

```bash
sudo tailscale up --exit-node=[tailscale IP of exit node] --exit-node-allow-lan-access=true --accept-routes
```

## Additional setup for Tailscale

<https://tailscale.com/kb/1320/performance-best-practices#ethtool-configuration>

```bash
NETDEV=$(ip -o route get 8.8.8.8 | cut -f 5 -d " ")
sudo ethtool -K $NETDEV rx-udp-gro-forwarding on rx-gro-list off
```

```bash
printf '#!/bin/sh\n\nethtool -K %s rx-udp-gro-forwarding on rx-gro-list off \n' "$(ip -o route get 8.8.8.8 | cut -f 5 -d " ")" | sudo tee /etc/networkd-dispatcher/routable.d/50-tailscale
sudo chmod 755 /etc/networkd-dispatcher/routable.d/50-tailscale
```

```bash
sudo /etc/networkd-dispatcher/routable.d/50-tailscale
test $? -eq 0 || echo 'An error occurred.'
```

## install NordVPN

Info: <https://support.nordvpn.com/hc/en-us/articles/20196094470929-Installing-NordVPN-on-Linux-distributions>

```bash
sh <(curl -sSf https://downloads.nordcdn.com/apps/linux/install.sh)
sudo usermod -aG nordvpn $USER
```

Generate a nordvpn token, then use it to login:

```bash
nordvpn login --token [nordvpn login token]
```

Change following settings:

```bash
nordvpn set firewall disabled
nordvpn set lan-discovery enabled
nordvpn set dns 1.1.1.1
nordvpn set autoconnect on singapore
```

Then

```bash
nordvpn c singapore
```

Test:

```bash
curl https://api.myip.com/
```

## Android

Make sure `Allow LAN access` is toggled off, otherwise not able to make use of exit node (no internet).
