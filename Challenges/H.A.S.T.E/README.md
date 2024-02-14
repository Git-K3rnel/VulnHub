# H.A.S.T.E

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.134	00:0c:29:b3:82:58	VMware, Inc.
192.168.127.254	00:50:56:f3:36:94	VMware, Inc.
```

The IP address is `192.168.127.134`, let's enumerate it


## 2.Enumeration

```bash
root@kali: nmap -p- -T4 -v 192.168.127.134

Nmap scan report for www.convert.me (192.168.127.134)
Host is up (0.0023s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 00:0C:29:B3:82:58 (VMware)
```

only port 80 is open, so let's navigate it
