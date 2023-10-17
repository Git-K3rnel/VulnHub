# Dina: 1.0.1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:d7:c0:71       PCS Systemtechnik GmbH
192.168.56.108  08:00:27:b9:c3:c6       PCS Systemtechnik GmbH
```

The IP address `192.168.56.108`, let's enumerate it.


## 2.Enumeration

```bash
root@kali: nmap -sV -sC -p- -oN nmap.out 192.168.56.108

Nmap scan report for 192.168.56.108
Host is up (0.00046s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-robots.txt: 5 disallowed entries 
|_/ange1 /angel1 /nothing /tmp /uploads
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Dina
MAC Address: 08:00:27:B9:C3:C6 (Oracle VirtualBox virtual NIC)
```

We explore the website and fuzz the web application :

![MainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/cd80ecb5-6813-4ff2-9c0b-adbe5f7088b6)

see robots.txt and the following directories are disallowed :

```text
User-agent: *
Disallow: /ange1
Disallow: /angel1
Disallow: /nothing
Disallow: /tmp
Disallow: /uploads
```

go to `nothing` directory and see the page source, there is a comment here :

```text
<!--
#my secret pass
freedom
password
helloworld!
diana
iloveroot
-->
```

we collect these possible passwords for later use, and now we can fuzz the application :

```bash
root@kali: gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.56.108

===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index                (Status: 200) [Size: 3618]
/uploads              (Status: 301) [Size: 318] [--> http://192.168.56.108/uploads/]
/secure               (Status: 301) [Size: 317] [--> http://192.168.56.108/secure/]
/robots               (Status: 200) [Size: 102]
/tmp                  (Status: 301) [Size: 314] [--> http://192.168.56.108/tmp/]
/nothing              (Status: 301) [Size: 318] [--> http://192.168.56.108/nothing/]
```












