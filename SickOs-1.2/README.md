# SickOs: 1.2

## 1.Get VM IP :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/8e84c627-dcce-4e28-9a2b-7f5309b7fc65)

The ip address is `192.168.127.230`, let's scan it.

## 2.Enumeration

```bash
Nmap scan report for 192.168.127.230
Host is up (0.00047s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 66:8c:c0:f2:85:7c:6c:c0:f6:ab:7d:48:04:81:c2:d4 (DSA)
|   2048 ba:86:f5:ee:cc:83:df:a6:3f:fd:c1:34:bb:7e:62:ab (RSA)
|_  256 a1:6c:fa:18:da:57:1d:33:2c:52:e4:ec:97:e2:9e:af (ECDSA)
80/tcp open  http    lighttpd 1.4.28
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: lighttpd/1.4.28
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:67:CE:D7 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Visiting the web application has nothing :
- no robots.txt
- no special directory
- no nikto special results
- no directory traversal vector
- no session management

  the only useful information from nikto is `/test` directory
