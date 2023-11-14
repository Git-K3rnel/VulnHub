# FOWSNIFF: 1

## 1.GET VM IP:

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/e385b77d-00dc-4c7e-94e2-61b73ecf1c5b)

The ip address is `192.168.56.111`, let's enumerate it.

## 2.Enumeration

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f80e208c-de6f-4236-afdb-bb479c08a047)

On port 80 there is nothing to check but the information it gives us :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/78aa10fd-fd01-41c4-9e37-51b77f080c77)

so i tried to search for `fowsniff corp` on google and i found a [twitter](https://twitter.com/fowsniffcorp?lang=en) page and a [github](https://raw.githubusercontent.com/berzerk0/Fowsniff/main/fowsniff.txt) page

that shows dump of passwords hashes, just use an online service or john to crack the hashes, after cracking them you will reach to this list :

```text
mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4=mailcall
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56=bilbo101
tegel@fowsniff:1dc352435fecca338acfd4be10984009=apples01
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb=skyler22
seina@fowsniff:90dc16d47114aa13671c697fd506cf26=scoobydoo2
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b=carp4ever
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11=orlando12
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e=07011972
```

now we can try to login to pop3 service using one of these credentials, just separate users and passwords in two file and brute force the pop3 :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/c2c3fcdc-a1eb-475a-be04-e8fae9fef567)

here i found a valid credential so let's login to mail server :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/21e99fb9-e927-40ec-8c46-c82fc2802865)

and let's check the content of the messages :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/62bb996a-b70d-44f2-89ba-c3b2a6708549)

in this message we find a possible ssh password, now we should test it again all users.

## 3.Gaining Shell

Again we try to brute force the ssh service with the newly found password and the list of users we created before :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/cfc5caf8-dea8-4ae2-a680-5ca0c6f34760)

yes, we found a credential for ssh login :

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/02565803-a334-4347-9fde-dcea5db429d1)

## 4.Privilege Escalation









