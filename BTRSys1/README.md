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








