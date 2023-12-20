# DC-3

## 1.Enumeration

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:0f	(Unknown: locally administered)
192.168.56.100	08:00:27:5f:5e:41	PCS Systemtechnik GmbH
192.168.56.115	08:00:27:31:0f:a8	PCS Systemtechnik GmbH
```

The IP address is `192.168.56.115`, let's scan it.

```bash
root@kali: nmap -sC -sV -p80 -v -oN nmap.out 192.168.56.115

Nmap scan report for 192.168.56.115
Host is up (0.00044s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 1194D7D32448E1F90741A97B42AF91FA
|_http-title: Home
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-generator: Joomla! - Open Source Content Management
MAC Address: 08:00:27:31:0F:A8 (Oracle VirtualBox virtual NIC)

```

The only open port is port 80, so we have to work on the web application:

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/8d992b5c-3a5d-4d38-898d-917b0ca7228d)

visiting the site, Wappalyzer shows this is a `Joomla` cms, also cheking the target with Nikto shows afew interesting directories :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/d70cfbff-32d9-4a05-b39c-3f9932602930)


























