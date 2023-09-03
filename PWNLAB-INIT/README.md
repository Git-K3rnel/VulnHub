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
