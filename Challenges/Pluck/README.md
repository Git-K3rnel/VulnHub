# Pluck

## 1.Scan Network

```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.133	00:0c:29:e8:d3:70	VMware, Inc.
192.168.127.254	00:50:56:eb:a0:df	VMware, Inc.
```

The IP address is `192.168.127.133`, let's enumerate it


## 2.Enumeration

```bash
root@kali: nmap -p22,80,3306,5355 -sV -sC -v -oN nmap.out 192.168.127.133

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.3p1 Ubuntu 1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e8:87:ba:3e:d7:43:23:bf:4a:6b:9d:ae:63:14:ea:71 (RSA)
|   256 8f:8c:ac:8d:e8:cc:f9:0e:89:f7:5d:a0:6c:28:56:fd (ECDSA)
|_  256 18:98:5a:5a:5c:59:e1:25:70:1c:37:1a:f2:c7:26:fe (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Pluck
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
3306/tcp open  mysql   MySQL (unauthorized)
5355/tcp open  llmnr?
MAC Address: 00:0C:29:E8:D3:70 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We first go for port 80 and check the website :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/b4b4d9d2-09c4-4692-84df-c265b67b5297)

checking different parts of the page i found `Contact Us` menu which loads the page like this :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/350d1de7-4dd5-451f-850e-115614ce36ca)

i used burp to repeat the process for a path traversal vulnerability and it worked :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2bc65341-4166-40f8-8ab7-e66e4c2ab9f8)

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/a65ba4fd-84bf-4435-96b2-ecd4abfebae4)













