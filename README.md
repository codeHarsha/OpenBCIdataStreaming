# Open BCI Data Streaming

This is a step by step process of how we can setup Open BCI, Raspberry Pi, Python, X code to stream Open BCI Data in the IOS App.

## Setup Hotspot for Raspberry Pi

Make sure that both Raspberry pi and your remote computer is connected to ethernet (same network). If your Raspberry/Remote computer is connected to WiFi this setup will fail, as the setup requires changes in network configurations.

SSH into the raspberry pi and follow below steps. firstly udpate and upgrade.

```bash
sudo apt-get update
sudo apt-get upgrade
```

To install the required packages, enter the following into the console

```bash
sudo apt-get -y install hostapd dnsmasq
```

Edit the dhcpcd file

```bash
sudo nano /etc/dhcpcd.conf
```

Scroll down, and at the bottom of the file, add

```bash
denyinterfaces wlan0
```

Open the interfaces file with the following command

```bash
sudo nano /etc/network/interfaces
```
your file should look like below

```bash
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

allow-hotplug wlan0
iface wlan0 inet static
      address 192.168.5.1
      netmask 255.255.255.0
      network 192.168.5.0
      broadcast 192.168.5.255

```

Configure Hostapd

```bash
sudo nano /etc/hostapd/hostapd.conf
```

Enter the following into that file. Feel fee to change the ssid (WiFi network name) and the wpa_passphrase (password to join the network) to whatever you'd like. You can also change the channel to something in the 1-11 range (if channel 6 is too crowded in your area).

```bash
interface=wlan0
driver=nl80211
ssid=piHotspot
hw_mode=g
channel=6
ieee80211n=1
wmm_enabled=1
ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_passphrase=12345678
rsn_pairwise=CCMP
```

hostapd does not know where to find this configuration file, so we need to provide its location to the hostapd startup script

```bash
sudo nano /etc/default/hostapd
```

Find the line #DAEMON_CONF="" and replace it with:

```bash
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

Configure Dnsmasq

```bash
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.bak
sudo nano /etc/dnsmasq.conf
```

In the created new dnsmasq.conf file, paste in the text below

```bash
interface=wlan0 
listen-address=192.168.5.1
bind-interfaces 
server=8.8.8.8
domain-needed
bogus-priv
dhcp-range=192.168.5.100,192.168.5.200,24h
```

Restart the Raspberry Pi using the following command

```bash
sudo reboot
```

After couple of minutes, In the nearby devices raspberry pi's hotspot will be broadcasted as the SSID you provided in the hostapd file.

Currently, raspberry will not be able to connect to WiFi as we have only one wlan interface in raspberry pi, we have used that interface for making it hotspot.

If we want to connect raspberry pi to WiFi, follow below steps

Before everything, you have to ssh into raspberry pi terminal, for that you will have to connect raspberry pi to LAN.


Step 1: edit the dhcpcd file.

```bash
sudo nano /etc/dhcpcd.conf
```

Scroll down, and at the bottom of the file, and comment denyinterfaces wlan0.

```bash
#denyinterfaces wlan0
```

Step 2: Make changes to /etc/network/interfaces file.

```bash
sudo nano /etc/network/interfaces
```
your file should look like below

```bash
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
    metric 300

allow-hotplug wlan0
iface wlan0 inet dhcp
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
    metric 200
```

if raspberry pi was previously configured to connect to particular wifi, you are good to go or else,
For connecting raspberry pi to wifi, you can add wifi details to  /etc/wpa_supplicant/wpa_supplicant.conf file or you can use "sudo raspi-config"

Restart the Raspberry Pi using the following command.

```bash
sudo reboot
```

Now, Raspberry Pi's hotspot will not be available and you can connect Raspberry Pi to WiFi.

## Developed by Harshavardhan Reddy Panyala

I will be working on future advancements to automate the process.
