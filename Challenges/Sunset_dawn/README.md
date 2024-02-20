# Sunset: dawn

## 1.GET VM IP

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:1d:88:c4       PCS Systemtechnik GmbH
192.168.56.122  08:00:27:e7:ae:cd       PCS Systemtechnik GmbH
```

The IP address is `192.168.56.122`, let's scan it.

## 2.Enumeration

```bash
root@kali: nmap -p80,139,445,3306 -sC -sV -oN nmap.out 192.168.56.122

PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL 5.5.5-10.3.15-MariaDB-1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.15-MariaDB-1
|   Thread ID: 12
|   Capabilities flags: 63486
|   Some Capabilities: SupportsCompression, Support41Auth, SupportsTransactions, IgnoreSigpipes, FoundRows, LongColumnFlag, Speaks41ProtocolOld, InteractiveClient, ConnectWithDatabase, Speaks41ProtocolNew, SupportsLoadDataLocal, ODBCClient, IgnoreSpaceBeforeParenthesis, DontAllowDatabaseTableColumn, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults|   Status: Autocommit
|   Salt: 5%[xm:vf}aPJc2nWF7$L
|_  Auth Plugin Name: mysql_native_password
MAC Address: 08:00:27:E7:AE:CD (Oracle VirtualBox virtual NIC)
Service Info: Host: DAWN

Host script results:
| smb2-time: 
|   date: 2024-02-20T07:44:44
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: dawn
|   NetBIOS computer name: DAWN\x00
|   Domain name: dawn
|   FQDN: dawn.dawn
|_  System time: 2024-02-20T02:44:44-05:00
|_clock-skew: mean: 1h39m57s, deviation: 2h53m12s, median: -2s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: DAWN, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
```

Checking on port 80 had nothing interesting, i started fuzzing and found these :

```bash
root@kali: gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.56.122

===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/logs                 (Status: 301) [Size: 315] [--> http://192.168.56.122/logs/]
/cctv                 (Status: 301) [Size: 315] [--> http://192.168.56.122/cctv/]
```

on `/logs` we see some system log files but only management.log is accessible :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c4d80524-cc60-4922-86e8-f95b3a03b4c6)














