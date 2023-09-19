# SickOs: 1.1

## 1.Get VM IP

```text
root㉿kali:~# arp-scan -I eth1 -l
                       
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.229 00:0c:29:25:b7:8a       VMware, Inc.
192.168.127.254 00:50:56:fd:64:8a       VMware, Inc.
```
The IP address is `192.168.127.229`, let's enumerate it.

## 2.Enumeration

```text
root㉿kali:~# nmap -sV -sC -p- -oN fullscan 192.168.127.229

Nmap scan report for 192.168.127.229
Host is up (0.00041s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 5.9p1 Debian 5ubuntu1.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 09:3d:29:a0:da:48:14:c1:65:14:1e:6a:6c:37:04:09 (DSA)
|   2048 84:63:e9:a8:8e:99:33:48:db:f6:d5:81:ab:f2:08:ec (RSA)
|_  256 51:f6:eb:09:f6:b3:e6:91:ae:36:37:0c:c8:ee:34:27 (ECDSA)
3128/tcp open   http-proxy Squid http proxy 3.1.19
|_http-server-header: squid/3.1.19
|_http-title: ERROR: The requested URL could not be retrieved
8080/tcp closed http-proxy
MAC Address: 00:0C:29:25:B7:8A (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

port `3128` is for squid proxy and is open, so the system is using some kind of proxy here

and port 8080 is closed, maybe because of proxy.

so we need to proxy our traffic through squid to see if any website is available, go [here](https://book.hacktricks.xyz/network-services-pentesting/3128-pentesting-squid) for details.

```text
root㉿kali:~# curl --proxy http://192.168.127.229:3128 http://192.168.127.229

<h1>
BLEHHH!!!
</h1>
```

yes, there is a website here, let's check for `robots.txt` :

```text
root㉿kali:~# curl --proxy http://192.168.127.229:3128 http://192.168.127.229/robots.txt

User-agent: *
Disallow: /
Dissalow: /wolfcms
```

the path `wolfcmd` is dissalowed here, i just set my firefox proxy to point to `192.168.127.229:3128` and visited the `/wolfcms` :

![mainpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/d1740ada-f57f-4b86-bed9-010d97eac250)

since this is a cms i quickly checked searchsploit for any exploit :

```text
searchsploit wolf cms  
----------------------------------------------------------------------------------
 Exploit Title                                          |  Path
----------------------------------------------------------------------------------
Wolf CMS - Arbitrary File Upload / Execution            | php/webapps/38000.txt
Wolf CMS 0.6.0b - Multiple Vulnerabilities              | php/webapps/15614.html
Wolf CMS 0.7.5 - Multiple Vulnerabilities               | php/webapps/18545.txt
Wolf CMS 0.8.2 - Arbitrary File Upload                  | php/webapps/36818.php
Wolf CMS 0.8.2 - Arbitrary File Upload (Metasploit)     | php/remote/40004.rb
Wolf CMS 0.8.3.1 - Remote Code Execution (RCE)          | php/webapps/51421.txt
----------------------------------------------------------------------------------
Shellcodes: No Results
```
i looked at one of [them](https://www.exploit-db.com/exploits/38000)


















