# GrimTheRipper: 1

## 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l

Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1   00:50:56:c0:00:01       VMware, Inc.
192.168.127.252 00:0c:29:b1:00:86       VMware, Inc.
192.168.127.254 00:50:56:f1:df:13       VMware, Inc.
```

The IP address is `192.168.127.252`, let's enumerate it

## 2.Enumeration

I started with checking and fuzzing the website and checking robots.txt:

```bash
root@kali: ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.127.252/FUZZ

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.127.252/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________
index                   [Status: 200, Size: 55, Words: 2, Lines: 4, Duration: 20ms]
image                   [Status: 200, Size: 67169, Words: 142, Lines: 220, Duration: 1ms]
index2                  [Status: 200, Size: 122, Words: 5, Lines: 12, Duration: 2ms]
robots                  [Status: 200, Size: 24, Words: 1, Lines: 4, Duration: 5ms]
```

we found a new directory `index2` which when you navigate to its page source you find this :



















