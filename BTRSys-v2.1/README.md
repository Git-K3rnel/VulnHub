#  BTRSys: v2.1

## 1.Get VM IP:

```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.235 00:0c:29:75:68:9d       VMware, Inc.
192.168.127.254 00:50:56:fd:64:8a       VMware, Inc.

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.026 seconds (126.36 hosts/sec). 3 responded
```

The IP address is `192.168.127.235`, let's scan it.
