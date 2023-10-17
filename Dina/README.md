# Dina: 1.0.1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:d7:c0:71       PCS Systemtechnik GmbH
192.168.56.108  08:00:27:b9:c3:c6       PCS Systemtechnik GmbH
```

The IP address `192.168.56.108`, let's enumerate it.


## 2.Enumeration

```bash
root@kali: nmap -sV -sC -p- -oN nmap.out 192.168.56.108

Nmap scan report for 192.168.56.108
Host is up (0.00046s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-robots.txt: 5 disallowed entries 
|_/ange1 /angel1 /nothing /tmp /uploads
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Dina
MAC Address: 08:00:27:B9:C3:C6 (Oracle VirtualBox virtual NIC)
```
