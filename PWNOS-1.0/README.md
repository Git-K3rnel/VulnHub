# PWNOS: 1.0

## Get VM IP
```bash
root@kali: nmap -sn 192.168.127.0/24

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-10 02:20 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00021s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.138
Host is up (0.00041s latency).
MAC Address: 00:0C:29:5E:18:C9 (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00033s latency).
MAC Address: 00:50:56:F4:78:0D (VMware)
Nmap scan report for 192.168.127.128
Host is up.
```

The IP address is `192.168.127.138`, let's enumerate the machine.

## Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.138 -oN fullscan

Nmap scan report for 192.168.127.138
Host is up (0.0019s latency).
Not shown: 995 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
| ssh-hostkey: 
|   1024 e4:46:40:bf:e6:29:ac:c6:00:e2:b2:a3:e1:50:90:3c (DSA)
|_  2048 10:cc:35:45:8e:f2:7a:a1:cc:db:a0:e8:bf:c7:73:3d (RSA)
80/tcp    open  http        Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.2.4 (Ubuntu) PHP/5.2.3-1ubuntu6
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
445/tcp   open  ��c��U      Samba smbd 3.0.26a (workgroup: MSHOME)
10000/tcp open  http        MiniServ 0.01 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
MAC Address: 00:0C:29:5E:18:C9 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.26a)
|   Computer name: ubuntuvm
|   NetBIOS computer name: 
|   Domain name: nsdlab
|   FQDN: ubuntuvm.NSDLAB
|_  System time: 2023-09-09T05:33:29-05:00
|_nbstat: NetBIOS name: UBUNTUVM, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: mean: 2h30m03s, deviation: 3h32m08s, median: 2s
|_smb2-time: Protocol negotiation failed (SMB2)
```

smb enumeration led to nothing intresting.

but exploring the web showed a web application :

![mainpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/4b436200-9be0-449e-aa5d-135320f2a4d0)

