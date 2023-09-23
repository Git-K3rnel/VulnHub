# MILNET: 1

## 1.GET VM IP

```text
root@kali: arp-scan -I eth1 -l
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       (Unknown)
192.168.127.231 00:0c:29:a0:3b:c4       (Unknown)
192.168.127.254 00:50:56:fd:64:8a       (Unknown)
```

the VM IP is `192.168.127.231`, let's enumerate it.

## 2.Enumeration
