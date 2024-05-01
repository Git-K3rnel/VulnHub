# Sumo

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.133	00:0c:29:6c:b0:85	VMware, Inc.
192.168.127.254	00:50:56:ea:34:a1	VMware, Inc.
```

The IP address is `192.168.127.133`, let's enumerate it.

### 2.Enumeration

```bash
root@kali: nmap -p- -sT -sV -sC -v -oN nmap.out 192.168.127.133
Nmap scan report for 192.168.127.133
Host is up (0.00033s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 06:cb:9e:a3:af:f0:10:48:c4:17:93:4a:2c:45:d9:48 (DSA)
|   2048 b7:c5:42:7b:ba:ae:9b:9b:71:90:e7:47:b4:a4:de:5a (RSA)
|_  256 fa:81:cd:00:2d:52:66:0b:70:fc:b8:40:fa:db:18:30 (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:6C:B0:85 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

try to scan port 80 with `nikto` :

```bash
root@kali: nikto -h 192.168.127.133
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/b0b2b5ff-8ca4-4de2-8e60-05778cf42758)











