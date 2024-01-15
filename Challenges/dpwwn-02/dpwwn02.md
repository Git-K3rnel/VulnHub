# dpwwn: 2

## 1.Enumeration
The ip address is `10.10.10.10`, let's scan it:

```bash
root@kali: nmap -v -p80,111,443,2049,33607,34141,47967,48661 -sC -sV -oN nmap.out 10.10.10.10

Nmap scan report for 10.10.10.10
Host is up (0.00041s latency).

PORT      STATE SERVICE  VERSION
80/tcp    open  http     Apache httpd 2.4.38 ((Ubuntu))
|_http-server-header: Apache/2.4.38 (Ubuntu)
|_http-title: dpwwn-02
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      45873/tcp6  mountd
|   100005  1,2,3      48661/tcp   mountd
|   100005  1,2,3      51220/udp   mountd
|   100005  1,2,3      52201/udp6  mountd
|   100021  1,3,4      34141/tcp   nlockmgr
|   100021  1,3,4      35639/tcp6  nlockmgr
|   100021  1,3,4      45437/udp6  nlockmgr
|   100021  1,3,4      55871/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
443/tcp   open  http     Apache httpd 2.4.38 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.38 (Ubuntu)
|_http-title: dpwwn-02
2049/tcp  open  nfs      3-4 (RPC #100003)
33607/tcp open  mountd   1-3 (RPC #100005)
34141/tcp open  nlockmgr 1-4 (RPC #100021)
47967/tcp open  mountd   1-3 (RPC #100005)
48661/tcp open  mountd   1-3 (RPC #100005)
```

i first start chekcing the nfs port and see the available shares:

```bash
root@kali: showmount -e 10.10.10.10

Export list for 10.10.10.10:
/home/dpwwn02 (everyone)
```

then mount it on the file system:

```bash
root@kali: mount -t nfs 10.10.10.10:/home/dpwwn02 /mnt/dpwwn
```





