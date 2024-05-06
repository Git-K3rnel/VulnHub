# Loly: 1

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l   
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.135	00:0c:29:0f:25:d7	VMware, Inc.
192.168.127.254	00:50:56:ea:34:a1	VMware, Inc.
```

The ip address is `192.168.127.135`, let's enumerate it

### 2.Enumeration
