Tested on Raspberry Pi 3B, running RaspberryPi OS. Linux raspberrypi 5.10.63-v7+ #1459 SMP Wed Oct 6 16:41:10 BST 2021 armv7l GNU/Linux

service-router ==================> wlan0 --------------------> wlan1 ==================> device

Install WIFI Adapter Driver(s)

```bash
sudo wget http://downloads.fars-robotics.net/wifi-drivers/install-wifi -O /usr/bin/install-wifi
sudo chmod +x /usr/bin/install-wifi
sudo install-wifi
```

Install required Packages

```bash
sudo apt install -y hostapd dnsmasq iptables-persistent
```


sudo nano /etc/dhcpcd.conf

	interface wlan0 ###
	static ip_address=192.168.10.1/24
	nohook wpa_supplicant
	
	
sudo nano /etc/udev/rules.d/72-persistent-net.rules

	ACTION=="add", SUBSYSTEM=="net", DRIVERS=="?*", ATTR{address}=="xx:xx:xx:xx:xx:xx", KERNEL=="w*",NAME="wlan1"
	ACTION=="add", SUBSYSTEM=="net", DRIVERS=="?*", ATTR{address}=="xx:xx:xx:xx:xx:xx", KERNEL=="w*",NAME="wlan0"

	
sudo nano /etc/dnsmasq.conf

	# disables dnsmasq reading any other files like /etc/resolv.conf for nameservers
	no-resolv
	# Interface to bind to
	interface=wlan0 ###
	except-interface=wlan1 ###
	except-interface=eth0
	# Specify starting_range,end_range,lease_time
	dhcp-range=192.168.10.16,192.168.10.64,255.255.255.0,6h
	# DNS addresses to send to the clients
	server=8.8.8.8
	server=8.8.4.4
	server=192.168.1.1
	# Logs
	log-facility=/var/log/dnsmasq.log
	log-queries
	# IP Reservation
	dhcp-host=f0:6e:0b:d3:00:8c,DESKTOP-R5HGJLG,192.168.10.200,infinite
	dhcp-host=20:16:b9:71:e9:db,BLRKEC1002491L,192.168.10.205,infinite
	dhcp-host=dc:a6:32:9a:27:28,192.168.10.240,infinite
	dhcp-host=52:64:2b:07:f1:8f,MiWiFi,192.168.10.2,infinite
	
cat /var/lib/misc/dnsmasq.leases
	
sudo nano /etc/sysctl.conf 

	net.ipv4.ip_forward=1
	net.ipv6.conf.all.forwarding=1
	
sudo nano /etc/hostapd/hostapd.conf

    #WITHOUT AUTH
	interface=wlan0 ###
	ssid=Raspberry_Free
	hw_mode=g
	channel=6
	auth_algs=1
	wmm_enabled=0

    #WITH AUTH
	interface=wlan0 ###
	driver=nl80211
	country_code=IN # important to set the region
	ssid=UniqueWLAN
	ieee80211d=1
	hw_mode=g
	channel=3
	ieee80211n=1
	wmm_enabled=1
	ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]
	macaddr_acl=0
	auth_algs=1
	ignore_broadcast_ssid=0
	wpa=2
	wpa_key_mgmt=WPA-PSK
	wpa_passphrase=DEC191991
	rsn_pairwise=CCMP

	
	

```
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/90_raspap.conf > /dev/null
sudo sysctl -p /etc/sysctl.d/90_raspap.conf
sudo /etc/init.d/procps restart

sudo iptables -t nat -A POSTROUTING -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 192.168.50.0/24 ! -d 192.168.50.0/24 -j MASQUERADE
sudo iptables-save | sudo tee /etc/iptables/rules.v4


sudo systemctl start dnsmasq.service	
sudo systemctl enable dnsmasq.service	
sudo systemctl unmask hostapd.service
sudo systemctl start hostapd.service
sudo systemctl enable hostapd.service
```
