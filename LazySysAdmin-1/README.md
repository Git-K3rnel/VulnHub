# LazySysAdmin: 1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:70:d2:8a       PCS Systemtechnik GmbH
192.168.56.106  08:00:27:e3:a8:a2       PCS Systemtechnik GmbH

3 packets received by filter, 0 packets dropped by kernel
```

The IP address is `192.168.56.106`, let's enumerate it.

## 2.Enumeration
