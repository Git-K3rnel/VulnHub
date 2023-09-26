![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/38870c8b-0936-492b-bd14-1062ee90c440)# VulnOS: 2

## 1.Get VM IP

```text
root@kali: arp-scan -I eth0 -l
Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:d6:4d:fa       PCS Systemtechnik GmbH
192.168.56.104  08:00:27:57:4f:aa       PCS Systemtechnik GmbH
```

The IP address is `192.168.56.104`, lets enumerate it.

## 2.Enumeration

```text
root@kali: nmap -sV -sC -p- -v -oN fullscan 192.168.56.104

Nmap scan report for 192.168.56.104
Host is up (0.00011s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 f5:4d:c8:e7:8b:c1:b2:11:95:24:fd:0e:4c:3c:3b:3b (DSA)
|   2048 ff:19:33:7a:c1:ee:b5:d0:dc:66:51:da:f0:6e:fc:48 (RSA)
|   256 ae:d7:6f:cc:ed:4a:82:8b:e8:66:a5:11:7a:11:5f:86 (ECDSA)
|_  256 71:bc:6b:7b:56:02:a4:8e:ce:1c:8e:a6:1e:3a:37:94 (ED25519)
80/tcp   open  http    Apache httpd 2.4.7 ((Ubuntu))
| http-methods:
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-title: VulnOSv2
|_http-server-header: Apache/2.4.7 (Ubuntu)
6667/tcp open  irc     ngircd
MAC Address: 08:00:27:57:4F:AA (Oracle VirtualBox virtual NIC)
Service Info: Host: irc.example.net; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

checking port 80 shows a basic website :

![mainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/378a7c15-24ab-4726-9a9e-d20150ac6047)

i first navigated each menu and its page source, `documentation` menu is weird and has nothing in UI but if you see the page source

or highlight the page, it mentions a directory (/jabcd0cs/) ;

![path](https://github.com/Git-K3rnel/VulnHub/assets/127470407/eb055fed-4dab-4632-b8c8-c9cdd5758e61)













