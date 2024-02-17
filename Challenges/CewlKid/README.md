# CewlKid: 1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:ac:dd:85       PCS Systemtechnik GmbH
192.168.56.121  08:00:27:d7:32:a1       PCS Systemtechnik GmbH
```

The ip address is `192.168.56.121`, let's scan it.


## 2.Enumeration

```bash
root@kali: nmap -p80,22,8080 -v -sC -sV -oN nmap.out 192.168.56.121

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7c:87:08:d7:ed:3a:9d:f6:be:91:7e:01:24:c8:74:c0 (RSA)
|   256 d6:22:68:01:2c:e2:c5:72:8e:1b:23:68:77:3e:10:7d (ECDSA)
|_  256 bb:91:85:49:86:fd:af:66:42:a3:f4:12:90:a3:31:6a (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-generator: Sitemagic CMS
|_http-title: Welcome - about us
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-favicon: Unknown favicon MD5: E6E1C631192DE24218AC9348293A46CB
```












