# Open BCI Data Streaming

****Please go through each section entirely before executing the process for first time setup. As we are dealing with network files, we need to be careful****

Existing Files in Our Raspberry Pi

1)	/home/fatemeh/automatehotspot/setup.sh -> for automation hotspot script.
2)	/home/fatemeh/automate-wifi/setup.sh -> for wifi reconfigure script.
3)	/home/fatemeh/stream/stream2.py -> for backend python flask server code.
4)	/etc/systemd/system/stream_app_instace2.service -> for autostart of backend.

Automation of Making Raspberry Pi a Hotspot Access Point

Steps:

1)	SSH into Raspberry Pi.

```bash
ssh fatemeh@eyecando.local
```

2)	Create a shell script file.

```bash
sudo nano setup.sh
```

3)	Copy the AutomateHotspot.sh script from code files.

4)	Paste the code into the shell script file you created.

5)	Give execute access to setup.sh

```bash
sudo chmod +x setup.sh
```

6)	Run the script

```bash
./setup.sh
```

7)	Reboot Pi

```bash
sudo reboot
```

Things to note:

1)	This process is only for the first time, from next time, you can just run the script and reboot.

2)	You can’t ssh into pi after running this script because, there is no internet access for raspberry pi.

3)	If you want to ssh into pi, connect pi to LAN.

4)	Then run the Reconfigure Wi-fi Automation script. That will connect to Wifi.

5)	The above reconfigure Wi-Fi automation is a manual process, you can do it from app as well.


Reconfigure Wi-fi Automation

Steps:

1)	Give LAN connection to Raspberry Pi.

2)	SSH into Raspberry Pi.

```bash
ssh fatemeh@eyecando.local
```

3)	Create a shell script file.

```bash
sudo nano wifireconfigure.sh
```

4)	Copy the ReconfigureWifi.sh script from code files.

5)	Paste the code into the shell script file you created.

6)	Give execute access to wifireconfigure.sh.

```bash
sudo chmod +x wifireconfigure.sh
```

7)	Run the script.

```bash
./ wifireconfigure.sh
```

8)	Reboot Pi.

```bash
sudo reboot
```

9)	Now, if you previously configured Wi-Fi, pi automatically connects to that Wi-Fi. If not, before rebooting, do

```bash
sudo raspi-config -> system settings -> Wireless -> give ssid and password.
```

Things to note:

1)	When you reboot, you can remove LAN.

2)	Raspberry Pi will no longer act as wifi hotspot.

3)	This is a manual process, you can also make raspberry pi reconfigure and connect to wifi directly from the open bci streaming ios app, using wifi communication button. This functionality is already implemented.

Services for Auto start of Backend When Raspberry Pi Turns ON

Steps:

1)	SSH into Raspberry Pi.

```bash
ssh fatemeh@eyecando.local
```

2)	cd into system directory

```bash
cd /etc/systemd/system
```

3)	create <your_desired_name>.service file

```bash
sudo nano <your_desired_name>.service
```

4)	Copy script from Strem_App.service and paste it to <your_desired_name>.service

5)	Run the service file for the first time.

```bash
sudo systemctl enable <your_desired_name>.service
sudo systemctl start <your_desired_name>.service
```

6)	To check the status

```bash
sudo systemctl status <your_desired_name>.service
```

Thing to note:

1)	This is only for the first time. Every time you turn on the raspberry pi, these files will automatically run.

2)	If you want to stop them.

```bash
sudo systemctl stop <your_desired_name>.service
```

3)	Make sure to change the path of the python file and virtual environment and user in the file, as per your details.

4)	In our existing raspberry pi, I created 2 instances of stream service, stream_app.service and stream_app_instace2.service

Backend Python Flask Server


APIs and their purpose:

1)	/api/startstream – starts bci stream instance.
2)	/api/stream – calling application will get stream data.
3)	/api/stopstream – stream is stopped.
4)	/api/connect_wifi – runs reconfigure wifi script and connects to SSID that user provided.
5)	/api/update_gaze – gets eye tracking data from calling application.


Important things to note:

1)	Please give correct path to reconfigure Wi-Fi script in the backend code.
2)	This backend code is for Cyton.
3)	This backend server will run as soon as the raspberry pi turns on.
The updated code is stream2.py in our raspberry pi and it will be hosted on ipaddress:5001 port. It uses stream_app_instace2.service for auto start.


iOS - Swift Code Setup

1)	The IP address that needs to be configured in swift code can be get from below Linux command “ifconfig”
2)	In the ifconfig output, look for wlan0: and get IP address beside inet.
3)	Make sure to do “pod install”, is you are encountering dependency issues.


Helpful Linux Commands

```bash
iwgetid -r  #to get the Wi-Fi SSID that pi currently connected to.
ifconfig #to get to know IP address for calling APIs.
```

This is a step by step process of how we can setup Raspberry Pi as hotspot manually.

## Setup Hotspot manually for Raspberry Pi

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
