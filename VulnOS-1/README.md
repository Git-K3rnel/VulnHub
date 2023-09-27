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

There are many ports open on this box, so we need alot of









