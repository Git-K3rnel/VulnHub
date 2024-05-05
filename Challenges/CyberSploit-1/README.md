# CYBERSPLOIT: 1

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.134	00:0c:29:95:0c:5f	VMware, Inc.
192.168.127.254	00:50:56:ea:34:a1	VMware, Inc.
```

The ip address is `192.168.127.134`, let's enumerate it

### 2.Enumeration

```bash
root@kali: nmap -sT -sV -sC -p- -oN nmap.out 192.168.127.134

Nmap scan report for www.convert.me (192.168.127.134)
Host is up (0.00037s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 01:1b:c8:fe:18:71:28:60:84:6a:9f:30:35:11:66:3d (DSA)
|   2048 d9:53:14:a3:7f:99:51:40:3f:49:ef:ef:7f:8b:35:de (RSA)
|_  256 ef:43:5b:d0:c0:eb:ee:3e:76:61:5c:6d:ce:15:fe:7e (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Hello Pentester!
MAC Address: 00:0C:29:95:0C:5F (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

check web page and see page source :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/53ac75f8-1570-4c3c-b7d0-e99ace8118e4)


we have username `itsskv`, now we check for `robots.txt` and we find a base64 string, let's decode:

```bash
root@kali: echo -n 'R29vZCBXb3JrICEKRmxhZzE6IGN5YmVyc3Bsb2l0e3lvdXR1YmUuY29tL2MvY3liZXJzcGxvaXR9' | base64 -d
Good Work !
Flag1: cybersploit{youtube.com/c/cybersploit}
```

i tried enumerating different things and i found nothing, then tried to brute force the SSH service with `rockyou.txt` and it didn not work

then i used flag1 ( `cybersploit{youtube.com/c/cybersploit}` ) as password and yes i connected to machine :

```bash
ssh itsskv@192.168.127.134
itsskv@192.168.127.134's password: 
Welcome to Ubuntu 12.04.5 LTS (GNU/Linux 3.13.0-32-generic i686)

 * Documentation:  https://help.ubuntu.com/

332 packages can be updated.
273 updates are security updates.

New release '14.04.6 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Your Hardware Enablement Stack (HWE) is supported until April 2017.
```








