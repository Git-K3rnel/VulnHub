# NULLBYTE: 1

## 1.Get VM IP

```text
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.232 00:0c:29:5a:43:e9       VMware, Inc.
192.168.127.254 00:50:56:fd:64:8a       VMware, Inc.
```

The IP address is `192.168.127.232`, let's enumerate the machine.

## 2.Enumeration

```text
root@kali: nmap -sV -sC -p- -v -oN fullscan 192.168.127.232

Nmap scan report for 192.168.127.232
Host is up (0.0022s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
| http-methods:
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Null Byte 00 - level 1
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          38099/tcp6  status
|   100024  1          49056/udp   status
|   100024  1          49267/tcp   status
|_  100024  1          53634/udp6  status
777/tcp   open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey:
|   1024 16:30:13:d9:d5:55:36:e8:1b:b7:d9:ba:55:2f:d7:44 (DSA)
|   2048 29:aa:7d:2e:60:8b:a6:a1:c2:bd:7c:c8:bd:3c:f4:f2 (RSA)
|   256 60:06:e3:64:8f:8a:6f:a7:74:5a:8b:3f:e1:24:93:96 (ECDSA)
|_  256 bc:f7:44:8d:79:6a:19:48:76:a3:e2:44:92:dc:13:a2 (ED25519)
49267/tcp open  status  1 (RPC #100024)
MAC Address: 00:0C:29:5A:43:E9 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

On port 80, we see a web page like this :

![mainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c2abb771-965c-4fbf-a5db-1906e0d2dc71)

fuzzing the website leads to finding a `phpmyadmin` path and nothing else, because there is no clue here i thought that there might be something in the image itself

so i downloaded the image (which is a gif file) and inspect it with `exiftool` :

```text
root@kali: exiftool main.gif

ExifTool Version Number         : 12.65
File Name                       : main.gif
Directory                       : .
File Size                       : 17 kB
File Modification Date/Time     : 2015:08:01 12:39:30-04:00
File Access Date/Time           : 2023:09:23 07:03:57-04:00
File Inode Change Date/Time     : 2023:09:23 06:52:11-04:00
File Permissions                : -rw-r--r--
File Type                       : GIF
File Type Extension             : gif
MIME Type                       : image/gif
GIF Version                     : 89a
Image Width                     : 235
Image Height                    : 302
Has Color Map                   : No
Color Resolution Depth          : 8
Bits Per Pixel                  : 1
Background Color                : 0
Comment                         : P-): kzMb5nVYJw
Image Size                      : 235x302
Megapixels                      : 0.071
```

as it is shows in `Comment` section we see a string, `kzMb5nVYJw`, which is more likely to be the path

by adding this string to the path in the website, page below is shown :


















