# Born2Root

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

check the file named `VDSoyuAXiO.txt` and you see a SSH private key here :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/5fc3cea2-3a53-49fe-91eb-95976e128380)

we can now check which user does this private key belongs to, i try user `martin` :


