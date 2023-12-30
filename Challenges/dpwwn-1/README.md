# dpwwn: 1

## 1.Enumeration

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:0f	(Unknown: locally administered)
192.168.56.119	08:00:27:d3:ce:c3	PCS Systemtechnik GmbH
192.168.56.100	08:00:27:29:ed:8e	PCS Systemtechnik GmbH
```

The ip address is `192.168.56.119`, let's scan it.


```bash
root@kali: nmap -v -p21,22,80,8080 -sV -sC -oN nmap.out 192.168.56.119

Nmap scan report for 192.168.56.119
Host is up (0.00024s latency).

PORT     STATE  SERVICE    VERSION
21/tcp   open   ftp        pyftpdlib 1.5.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--   1 root     root         1062 Jul 29  2019 backup
| ftp-syst: 
|   STAT: 
| FTP server status:
|  Connected to: 192.168.56.119:21
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
22/tcp   open   ssh        OpenSSH 7.9p1 Debian 10 (protocol 2.0)
| ssh-hostkey: 
|   2048 71:bd:fa:c5:8c:88:7c:22:14:c4:20:03:32:36:05:d6 (RSA)
|   256 35:92:8e:16:43:0c:39:88:8e:83:0d:e2:2c:a4:65:91 (ECDSA)
|_  256 45:c5:40:14:49:cf:80:3c:41:4f:bb:22:6c:80:1e:fe (ED25519)
80/tcp   closed http
8080/tcp closed http-proxy
MAC Address: 08:00:27:D3:CE:C3 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```