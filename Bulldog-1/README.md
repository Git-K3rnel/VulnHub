# Bulldog: 1

## 1.Get VM IP:

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:3c:59:b7       PCS Systemtechnik GmbH
192.168.56.107  08:00:27:38:4c:8d       PCS Systemtechnik GmbH
```

The IP address is `192.168.56.107`, let's scan it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC -p- -oN nmap.out 192.168.56.107

Nmap scan report for 192.168.56.107
Host is up (0.00050s latency).

PORT     STATE SERVICE VERSION
23/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 20:8b:fc:9e:d9:2e:28:22:6b:2e:0e:e3:72:c5:bb:52 (RSA)
|   256 cd:bd:45:d8:5c:e4:8c:b6:91:e5:39:a9:66:cb:d7:98 (ECDSA)
|_  256 2f:ba:d5:e5:9f:a2:43:e5:3b:24:2c:10:c2:0a:da:66 (ED25519)
80/tcp   open  http    WSGIServer 0.1 (Python 2.7.12)
|_http-title: Bulldog Industries
|_http-server-header: WSGIServer/0.1 Python/2.7.12
8080/tcp open  http    WSGIServer 0.1 (Python 2.7.12)
|_http-title: Bulldog Industries
MAC Address: 08:00:27:38:4C:8D (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

port `23` is used for ssh and we now visit the web application :

![mainpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c08d7dd9-9063-48ab-8cad-6b6738e1844f)

we try to fuzz the directories :

```bash
root@kali: gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.56.107

===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 0] [--> http://192.168.56.107/admin/]
/dev                  (Status: 301) [Size: 0] [--> http://192.168.56.107/dev/]
/notice               (Status: 301) [Size: 0] [--> http://192.168.56.107/notice/]
```

visiting the `/admin` page shows a django admin panel login, but we need username and password:

![admin](https://github.com/Git-K3rnel/VulnHub/assets/127470407/13a0ccf5-917f-4ec7-9a9b-0cbdcdde5fdb)

so we go to `/dev` : 

![dev](https://github.com/Git-K3rnel/VulnHub/assets/127470407/98c856d8-e169-4fc9-9b2c-2daf686fabde)

here we see afew users mentioned at the bottom of the page, but if we see the page source we find user passwords hashes too :






