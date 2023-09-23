# MILNET: 1

## 1.GET VM IP

```text
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       (Unknown)
192.168.127.231 00:0c:29:a0:3b:c4       (Unknown)
192.168.127.254 00:50:56:fd:64:8a       (Unknown)
```

the VM IP is `192.168.127.231`, let's enumerate it.

## 2.Enumeration

```text
root@kali: nmap -sV -sC 192.168.127.231 -p- -v -oN fullscan

Nmap scan report for 192.168.127.231
Host is up (0.0018s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 9b:b5:21:38:96:7f:85:bd:1b:aa:9a:70:cf:db:cd:36 (RSA)
|   256 93:30:be:c2:af:dd:81:a8:25:2b:57:e5:01:49:91:57 (ECDSA)
|_  256 37:40:2b:cc:27:ae:89:22:d0:d2:65:65:c4:9b:53:42 (ED25519)
80/tcp open  http    lighttpd 1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: lighttpd/1.4.35
MAC Address: 00:0C:29:A0:3B:C4 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

lets' see the web page :

![mainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/1d8bfdcd-6cef-4e90-8159-46237d681b72)

analyzing the website shows that it is using basic functions and have basic pages, 



















