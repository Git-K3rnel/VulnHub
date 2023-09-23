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

analyzing the website shows that it is using basic functions and have basic pages, but if you look closer you see a parameter

that is used to call each page, `route`, which opens new pages :


![route](https://github.com/Git-K3rnel/VulnHub/assets/127470407/7fb366da-a6bd-4447-85c8-1c60876b3dea)

since this is a parameter that loads other pages, i decided to first FUZZ any other pages available by this parameter :

```text
root@kali: ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -H "Content-Type: application/x-www-form-urlencoded" -X POST -d "route=FUZZ" -u http://192.168.127.231/content.php -fs 1

[Status: 200, Size: 146, Words: 7, Lines: 10, Duration: 13ms]
    * FUZZ: index

[Status: 200, Size: 110, Words: 8, Lines: 6, Duration: 14ms]
    * FUZZ: main

[Status: 200, Size: 64312, Words: 2958, Lines: 703, Duration: 12ms]
    * FUZZ: info

[Status: 200, Size: 533, Words: 31, Lines: 16, Duration: 12ms]
    * FUZZ: nav

[Status: 500, Size: 369, Words: 37, Lines: 12, Duration: 609ms]
    * FUZZ: content

[Status: 200, Size: 3902, Words: 519, Lines: 30, Duration: 7ms]
    * FUZZ: bomb

[Status: 200, Size: 254, Words: 26, Lines: 14, Duration: 6ms]
    * FUZZ: props
```

yes, i found `info` directory and navigated to it using firefox `edit and resend`:

![php](https://github.com/Git-K3rnel/VulnHub/assets/127470407/38b760ba-2a44-4598-aea3-b1390c349ab3)

it shows phpinfo which is alot useful, pay attention to this section :

![urlopen](https://github.com/Git-K3rnel/VulnHub/assets/127470407/74d321a3-7aa5-4f23-ad53-aa2d0c3425e6)

this line shows that we can include external URLs and it loads it, meaning we have `RFI` here, the `route`

parameter is interesting because it suspicious to `LFI` too :

![LFI](https://github.com/Git-K3rnel/VulnHub/assets/127470407/25f01221-8a2c-45b0-87a8-9f48772b0242)

as you can see, it loads the info page again. so since we have LFI and RFI here, if first tried to load the

source code of the pages with PHP wrapper base64 encode :

![wrapper](https://github.com/Git-K3rnel/VulnHub/assets/127470407/1483dd4b-0c48-45ab-a622-bb0bbe4d3040)

but it did not show anything, i tried to use RFI and used pentest monkey php reverse shell and saved it to `revshell.php`

and using python http server i sent a request to my server to get a reverse shell














