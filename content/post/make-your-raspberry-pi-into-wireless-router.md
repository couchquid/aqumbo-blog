+++
date = "2017-11-02T22:47:50+02:00"
title = "Make your Raspberry Pi 3 into a wireless router"
type = "post"
+++

### Setup

You need:
1x Raspberry Pi 3
1x microUSB Power cable with 2.5A
1x microSD 8gb Class 10
As many USB Ethernet adapters as you'll like.

Install raspian on the SD-card.

Change the default password.
```
$ passwd
```

Update the system.
```
$ sudo apt update
$ sudo apt dist-upgrade
$ sudo rpi-update
```

Start the SSH daemon.
```
$ sudo nano /etc/ssh/sshd_config
```
```
Port 22221
```
```
$ sudo systemctl enable ssh
```

Install what we need.
```
$ sudo apt install dnsmasq hostapd bridge-utils
```

Set up the Wifi
```
$ sudo nano /etc/hostapd/hostapd.conf
```
```
interface=wlan0
bridge=br0
driver=nl80211
ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]
wmm_enabled=1
ssid=YourWifiNetworkName
hw_mode=g
channel=6
ieee80211n=1
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_passphrase=YourWifiPassword
rsn_pairwise=CCMP
```

```
$ sudo nano /etc/network/interfaces
```
```
auto lo eth0 wlan0
iface lo inet loopback

iface eth0 inet dhcp
iface wlan0 inet manual

iface br0 inet static
    bridge_fd 0
    address 10.0.100.1
    netmask 255.255.255.0
    network 10.0.100.0
    broadcast 10.0.100.255
    bridge_ports eth1 eth2
    bridge_hw <hardware address of eth0>
    hostapd /etc/hostapd/hostapd.conf
```

### Configure DHCP
Will assign addresses from 10.0.100.100 to 10.0.100.200

```
$ sudo nano /etc/dnsmasq.conf
```
```
interface=br0
dhcp-range=private,10.0.100.100,10.0.100.200,12h

domain-needed
bogus-priv

dhcp-option=42,0.0.0.0

dhcp-option=vendor:MSFW,2,1i

dhcp-lease-max=506

server=208.67.222.222
```

```
$ sudo nano /etc/sysctl.conf
```
```
net.ipv4.ip_forward=1
```

```
$ sudo nano /etc/network/if-pre..up/iptables
```
```
#!/bin/bash

/sbin/iptables-restore < /etc/iptables.up.rules
```

```
$ sudo chmod +x /etc/network/ip-pre.up/iptables
```
```
$ sudo iptables -t nat -A POSTRUTING -o eth0 -j MASQUERADE
$ sudo iptables -A FORWARD -i br0 -o eth0 -j ACCEPT
```
```
$ sudo iptables-save > /etc/iptables.up.rules
```
```
$ sudo nano /etc/rc.local
```
Should look like this.
```
# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi

/sbin/ifup br0

exit 0
```

```
sudo systemctl enable dnsmasq
```

Need to fix an issue with dhcpcd in raspian stretch.

```
$ sudo nano /usr/lib/dhcpcd5/dhcpcd
```
```
#!/bin/sh -e

#
# This file belongs in /usr/lib/dhcpcd5/dhcpcd how you get it there is up to you
#

DHCPCD=/sbin/dhcpcd
INTERFACES=/etc/network/interfaces
REGEX="^[[:space:]]*iface[[:space:]](*.*)[[:space:]]*inet[[:space:]]*(dhcp|static)"
EXCLUDES=""

if grep -q -E $REGEX $INTERFACES; then
    #echo "Not running dhcpcd because $INTERFACES"
    #echo "defines some interfaces that will use a"
    #echo "DHCP client or static address"
    #exit 6
    for iface in `grep -E $REGEX $INTERFACES | cut -f2 -d" "`
    do
        if [[ $EXCLUDES != "" ]]; then
            EXCLUDES="${EXCLUDES}|${iface}"
        else
            EXCLUDES="${iface}"
        fi
    done
    EXCLUDES="(${EXCLUDES})"
fi

exec $DHCPCD -Z $EXCLUDES $@
```
