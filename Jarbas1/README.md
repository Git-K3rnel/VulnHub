# Jarbas: 1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.240 00:0c:29:e1:f2:ed       VMware, Inc.
192.168.127.254 00:50:56:e5:26:19       VMware, Inc.

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.055 seconds (124.57 hosts/sec). 3 responded
```

The IP address is `192.168.127.240`

## 2.Enumeration

```bash
root@kali: nmap -sV -sC -p- -v -oN nmap.out 192.168.127.240

Nmap scan report for 192.168.127.240
Host is up (0.00088s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 28:bc:49:3c:6c:43:29:57:3c:b8:85:9a:6d:3c:16:3f (RSA)
|   256 a0:1b:90:2c:da:79:eb:8f:3b:14:de:bb:3f:d2:e7:3f (ECDSA)
|_  256 57:72:08:54:b7:56:ff:c3:e6:16:6f:97:cf:ae:7f:76 (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
| http-methods: 
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-title: Jarbas - O Seu Mordomo Virtual!
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
3306/tcp open  mysql   MariaDB (unauthorized)
8080/tcp open  http    Jetty 9.4.z-SNAPSHOT
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
|_http-favicon: Unknown favicon MD5: 23E8C7BD78E8CD826C5A6073B15068B1
MAC Address: 00:0C:29:E1:F2:ED (VMware)
```

We immediately explore port 80 and 8080, in port 80 there is just a normal website that leads to nothing but

port 8080 serves a jenkins server with a login form. investigating internet to find the version of jenkin i found

this [guide](https://cloud.hacktricks.xyz/pentesting-ci-cd/jenkins-security) which says navigating to `/oops` shows the version

of jenkins so :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/a3345335-5afc-4c3e-9bc3-9549de8f0c8c)
![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/d15f5741-2f13-4c0a-b840-e274631e23b9)
















