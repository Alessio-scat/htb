# Writeup Preignition

#Custom Applications
#Apache
#Reconnaissance
#Web Site Structure Discovery
#Default Credentials

## Introduction


## Enumeration 

- `nmap -sV -sC {TARGET_IP}`
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-01 04:51 EDT
Nmap scan report for 10.129.205.209
Host is up (0.13s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome to nginx!

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 648.47 seconds
```

- `gobuster --dir {URl} --wordlist {path} -x php`
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.205.209/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin.php            (Status: 200) [Size: 999]
/admin.php            (Status: 200) [Size: 999]
Progress: 9228 / 9230 (99.98%)
===============================================================
Finished
===============================================================
```

## FootHold

go in the {URL}/admin.php

username: admin
password: admin

flag: 6483bee07c1c1d57f14e5b0717503c73