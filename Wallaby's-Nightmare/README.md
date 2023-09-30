# Wallaby's: Nightmare

## 1.GET VM IP

```text
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.233 00:0c:29:96:ab:45       VMware, Inc.
192.168.127.254 00:50:56:fd:64:8a       VMware, Inc.
```

The IP address is `192.168.127.233`, let's enumerate the machine.

## 2.Enumeration

```text
root@kali: nmap -sV -sC -p- -v -oN fullscan 192.168.127.233

Nmap scan report for 192.168.127.233
Host is up (0.00012s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 6e:07:fc:70:20:98:f8:46:e4:8d:2e:ca:39:22:c7:be (RSA)
|   256 99:46:05:e7:c2:ba:ce:06:c4:47:c8:4f:9f:58:4c:86 (ECDSA)
|_  256 4c:87:71:4f:af:1b:7c:35:49:ba:58:26:c1:df:b8:4f (ED25519)
80/tcp   open     http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Wallaby's Server
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
6667/tcp filtered irc
MAC Address: 00:0C:29:96:AB:45 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

port `6667` is filtered here, it's weird, let's begin by checking the website :

![firstPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/9044b9e0-6133-4d36-adb8-1aa1411d4985)

i entered my name and clicked submit :

![secondPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/382e8f34-5d08-4879-ae79-116ff0617291)

in the next page there is a message, but look at the URL and the page parameter which can be manipulated

i entered another word in `page` parameter and the page below showed up :

![testPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/12e9fad5-4921-40e0-81d2-d7bc1a0d9b8a)

now i'm suspicious to this parameter, i tried path traversal payloads and it worked :

![pathTraversal](https://github.com/Git-K3rnel/VulnHub/assets/127470407/661ac9e0-e246-4ff4-96bf-dd340e447687)

we found three users here :

```text
walfin:x:1000:1000:walfin,,,:/home/walfin:/bin/bash
steven?:x:1001:1001::/home/steven?:/bin/bash
ircd:x:1003:1003:,,,:/home/ircd:/bin/bash
```

then i tried to see other files like `/etc/shadow` that this message appeared :

![shadowFile](https://github.com/Git-K3rnel/VulnHub/assets/127470407/4a39972b-2181-4e13-bec2-7d313a7ed7c1)

after this i could not connect to machine on port 80 any more, i think diffucly increase is that the service is not 

available on port 80, so i scanned the machine one more time :

```text
root@kali: nmap -sV -sC  -p- -v -oN fullscan2 192.168.127.233

Nmap scan report for 192.168.127.233
Host is up (0.00013s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 6e:07:fc:70:20:98:f8:46:e4:8d:2e:ca:39:22:c7:be (RSA)
|   256 99:46:05:e7:c2:ba:ce:06:c4:47:c8:4f:9f:58:4c:86 (ECDSA)
|_  256 4c:87:71:4f:af:1b:7c:35:49:ba:58:26:c1:df:b8:4f (ED25519)
6667/tcp  filtered irc
60080/tcp open     http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Wallaby's Server
|_http-server-header: Apache/2.4.18 (Ubuntu)
MAC Address: 00:0C:29:96:AB:45 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

yes, the service is now has been moved on port 60080, so let's open the site on this port :

![newPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/ad7c2230-6008-43b3-84ea-83c93f0719d5)








