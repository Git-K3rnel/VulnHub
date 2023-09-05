# FRISTILEAKS: 1.3

## 1.Get VM IP

```bash
root@kali: nmap -sn 192.168.127.0/24

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-05 01:18 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00018s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.135
Host is up (0.00056s latency).
MAC Address: 08:00:27:A5:A6:76 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.127.254
Host is up (0.00049s latency).
MAC Address: 00:50:56:F4:78:0D (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.07 seconds
```

The IP address is `192.168.127.135`, let scan it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.135 -oN nmapresult

Nmap scan report for 192.168.127.135
Host is up (0.00025s latency).
Not shown: 989 filtered tcp ports (no-response), 10 filtered tcp ports (host-prohibited)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.15 ((CentOS) DAV/2 PHP/5.3.3)
|_http-server-header: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
| http-robots.txt: 3 disallowed entries 
|_/cola /sisi /beer
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
MAC Address: 08:00:27:A5:A6:76 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.12 seconds
```
The only open port here is `80`, from the nmap report we see `robots.txt` has 3 routes

navigating to these directories shows the same content :

![page](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2d1e90c6-1d88-4740-8469-f6c61cdbe270)

i started fuzzing the application with different methods but found nothing except `/images/` and `/icons/` directory that had nothing intersting.

back to the main page saying "KEEP CALM AND DRINK FRISTI", the word `fristi` was weird for me so tried navigating to `fristi` :

![fristipage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/86780385-6f4b-4857-9104-30b6d7043aea)

yes, we found a login page, i tried some sql injection payloads but did not work.

viewing the page source :

![pagesource](https://github.com/Git-K3rnel/VulnHub/assets/127470407/8cc548d1-f8b8-434e-aa7d-2e33b2f66633)

the comment here is by `eezeepz`, maybe a username ? write it down.

scrolling down a little shows base64 data image :


![datablob](https://github.com/Git-K3rnel/VulnHub/assets/127470407/606d6e18-a4a9-46d1-984b-e566391651c3)

convert it back to image using an online service or browser itself :

![kekek](https://github.com/Git-K3rnel/VulnHub/assets/127470407/bfffc291-b028-4751-9678-e301cc6def0d)

using `eezeepz` as user and the string in the picture `keKkeKKeKKeKkEkkEk` you can login to website :

![login](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5b34ac61-581a-42ee-b6bf-0c18c48db81a)

here we see an upload functionality that only allows uploading `png,jpg,gif` files, after trying afew methods to bypass it

i found that it checks only the last extention of a file, for example `test.php.png` is acceptable. so i uploaded a php reverse shell

and navigated to `/uploads/test.php.png` directory





