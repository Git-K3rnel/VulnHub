# Boss Player CTF

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l
Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:12	(Unknown: locally administered)
192.168.56.100	08:00:27:16:f6:de	PCS Systemtechnik GmbH
192.168.56.123	08:00:27:de:8c:54	PCS Systemtechnik GmbH
```

let's scan the ip address `192.168.56.123`

### 2.Enumeration

