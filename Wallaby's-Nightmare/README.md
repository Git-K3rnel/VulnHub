# Wallaby's: Nightmare

## 1.GET VM IP

```text
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.233 00:0c:29:96:ab:45       VMware, Inc.
192.168.127.254 00:50:56:fd:64:8a       VMware, Inc.
```

The IP address is `192.168.127.233`, let's enumerate the machine.

## 2.Enumeration
