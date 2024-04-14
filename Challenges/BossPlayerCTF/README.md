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

```bash
root@kali:nmap -sT -sV -v -p- -oN nmap.out 192.168.56.123
Host is up (0.00060s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
MAC Address: 08:00:27:DE:8C:54 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

check web page :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/53d19546-781b-4327-ba96-1a53c7cd6d75)

check source page:

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2d30e9b0-1ca1-4b34-bfe0-d91c54c69ae9)
