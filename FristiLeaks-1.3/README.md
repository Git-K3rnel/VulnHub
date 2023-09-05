# FRISTILEAKS: 1.3

## 1.Get VM IP

```bash
root@kali: nmap -sn 192.168.127.0/24

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-05 01:18 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00018s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.135
Host is up (0.00056s latency).
MAC Address: 08:00:27:A5:A6:76 (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.127.254
Host is up (0.00049s latency).
MAC Address: 00:50:56:F4:78:0D (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.07 seconds
```

The IP address is `192.168.127.135`, let scan it.

## 2.Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.135 -oN nmapresult
```
