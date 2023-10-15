#  BTRSys: v2.1

## 1.Get VM IP:

```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.235 00:0c:29:75:68:9d       VMware, Inc.
192.168.127.254 00:50:56:fd:64:8a       VMware, Inc.

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.026 seconds (126.36 hosts/sec). 3 responded
```

The IP address is `192.168.127.235`, let's scan it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC -oN fullscan -p- 192.168.127.235

Nmap scan report for 192.168.127.235
Host is up (0.00073s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.127.128
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 08:ee:e3:ff:31:20:87:6c:12:e7:1c:aa:c4:e7:54:f2 (RSA)
|   256 ad:e1:1c:7d:e7:86:76:be:9a:a8:bd:b9:68:92:77:87 (ECDSA)
|_  256 0c:e1:eb:06:0c:5c:b5:cc:1b:d1:fa:56:06:22:31:67 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_Hackers
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:75:68:9D (VMware)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
Port 21 has nothing to do with. so we start fuzzing on port 80 ;

```bash
root@kali: gobuster dir --url http://192.168.127.235 --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

/upload               (Status: 301) [Size: 319] [--> http://192.168.127.235/upload/]
/wordpress            (Status: 301) [Size: 322] [--> http://192.168.127.235/wordpress/]
/javascript           (Status: 301) [Size: 323] [--> http://192.168.127.235/javascript/]
/INSTALL              (Status: 200) [Size: 1241]
/LICENSE              (Status: 200) [Size: 1672]
/COPYING              (Status: 200) [Size: 35147]
/CHANGELOG            (Status: 200) [Size: 224]
```

so we have wordpress here, we can enumerate it with `wpscan`



