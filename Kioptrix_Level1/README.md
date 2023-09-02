# Kioptrix Level 1

### 1.Get VM IP
Use `nmap` to find the ip address of the machine :

```text
root@kali:~# nmap -sn 192.168.127.0/24

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-02 07:41 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00016s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.129
Host is up (0.00026s latency).
MAC Address: 00:0C:29:E3:0C:6A (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00019s latency).
MAC Address: 00:50:56:F4:78:0D (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.03 seconds
```

### 2.Enumeration

So the ip address is `192.168.127.129`, we try to scan the machine for open ports :


```text
root@kali~# nmap -sV -sC 192.168.127.129 -oN full_scan

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-02 07:44 EDT
Nmap scan report for 192.168.127.129
Host is up (0.0016s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 2.9p2 (protocol 1.99)
| ssh-hostkey: 
|   1024 b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86 (RSA1)
|   1024 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71 (DSA)
|_  1024 ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81 (RSA)
|_sshv1: Server supports SSHv1
80/tcp   open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
|_http-title: Test Page for the Apache Web Server on Red Hat Linux
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| http-methods: 
|_  Potentially risky methods: TRACE
111/tcp  open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1           1024/tcp   status
|_  100024  1           1024/udp   status
139/tcp  open  netbios-ssn Samba smbd (workgroup: ORMYGROUP)
443/tcp  open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-09-26T09:32:06
|_Not valid after:  2010-09-26T09:32:06
|_ssl-date: 2023-09-02T11:46:27+00:00; +1m53s from scanner time.
|_http-title: 400 Bad Request
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_RC4_64_WITH_MD5
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
1024/tcp open  status      1 (RPC #100024)
MAC Address: 00:0C:29:E3:0C:6A (VMware)

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: 1m52s
```

as we can see the interesting ports are `139` and `80`.

on port 80 there is a test page of apache but let's enumerate port 139 with `metasploit` :


![msfconsole](https://github.com/Git-K3rnel/VulnHub/assets/127470407/e04f5491-f079-492a-9ac2-2c8d21be8bc2)

version of samba is `2.2.1a`, i use `searchsploit` to find any exploit for this version :

```bash
searchsploit samba 2.2.1a
```

![searchsploit](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5bf347cd-9c35-4eb2-8a2d-baec00b797c5)


### 3.Gaining Shell


Try the first finding `trans2open Overflow (Metasploit)` which the version matches our samaba version.

again we use metasploit for this one, use `linux/samba/trans2open` as the exploit and `generic/shell_reverse_tcp` as the payload

set `rhost` to victim IP and `lhost` to attacking machine IP and type `run`:

![exploit](https://github.com/Git-K3rnel/VulnHub/assets/127470407/6f77fca6-d21a-46a7-85bc-b54bf6b6f0b2)

and boom, you are root now :)















