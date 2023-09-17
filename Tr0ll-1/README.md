# Tr0ll: 1

## 1.Get VM IP

```bash
root@kali: nmap -sn 192.168.127.0/24

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-17 06:57 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00042s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.141
Host is up (0.00032s latency).
MAC Address: 00:0C:29:1C:FD:35 (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00037s latency).
MAC Address: 00:50:56:EF:E6:A0 (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.60 seconds
```

The ip address is `192.168.127.141`, let's enumerate it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.141 -oN scan

Nmap scan report for 192.168.127.141
Host is up (0.00063s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.127.128
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 600
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap [NSE: writeable]
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d6:18:d9:ef:75:d3:1c:29:be:14:b5:2b:18:54:a9:c0 (DSA)
|   2048 ee:8c:64:87:44:39:53:8c:24:fe:9d:39:a9:ad:ea:db (RSA)
|   256 0e:66:e6:50:cf:56:3b:9c:67:8b:5f:56:ca:ae:6b:f4 (ECDSA)
|_  256 b2:8b:e2:46:5c:ef:fd:dc:72:f7:10:7e:04:5f:25:85 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/secret
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.7 (Ubuntu)
MAC Address: 00:0C:29:1C:FD:35 (VMware)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

visit `/robots.txt` of the web page and it shows a disallowed directory :

```text
User-agent:*
Disallow: /secret
```

visiting the above directory has nothing interesting, we take a look at port `21` for `anonymous` login :

```bash
root@kali: ftp 192.168.127.141

Connected to 192.168.127.141.
220 (vsFTPd 3.0.2)
Name (192.168.127.141:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||54900|).
150 Here comes the directory listing.
-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap
226 Directory send OK.
ftp> 
```

we find from here that file `lol.pcap` is available to download, so download and open it with wireshark

filter for `ftp-data` and you will find the packet wihch contains the secret directory :

![wireshark](https://github.com/Git-K3rnel/VulnHub/assets/127470407/556e5be5-5371-4f56-aafc-d4f58dff2da1)

























