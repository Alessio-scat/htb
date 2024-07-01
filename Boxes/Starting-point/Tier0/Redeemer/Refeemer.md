# Detailed description of the "Redeemer Write-up" document process

## Introduction

Databases are collections of organized information that can be easily accessed, managed and updated. In most environments, database systems are very important as they communicate information related to your sales transactions, product inventory, customer profiles and marketing activities.

There are different types of database, and one of them is Redis, which is an in-memory database. In-memory databases are those that essentially rely on main memory for data storage (i.e., the database is managed in the system's RAM), as opposed to databases that store data on disk or SSDs. As main memory is much faster than secondary memory, data retrieval times in the case of in-memory databases are very short, offering highly efficient and minimal response times.

In-memory databases like Redis are generally used to cache frequently requested data for rapid retrieval. For example, if a website displays prices on its home page, the site may first check to see if the required prices are in Redis; if not, it checks the traditional database (such as MySQL or MongoDB). When the value is loaded from the database, it is then stored in Redis for a shorter period (seconds, minutes or hours), in order to process all similar requests arriving during that period. For a site with a lot of traffic, this configuration allows much faster retrieval for the majority of requests, while still having stable long-term storage in the main database.

This lab focuses on enumerating a remote Redis server and then flushing its database to recover the flag. In the process, we learn how to use the `redis-cli` command-line utility, which helps to interact with the Redis service. We also learn some basic `redis-cli` commands, used to interact with the Redis server and the key-value database.

## Introduction to Pentesting

### 1. Enumeration stage
**Objective**: Document the current state of the target to learn as much as possible.
- Use `nmap` to scan open ports and determine which services are running.

## Detailed enumeration

### 1. Ping the target
- Use `ping {IP_of_the_target}` to check the connection. Stop with CTRL+C after two successful responses.

### 2. Scan open ports
- Command**: `nmap -sV -p- {IP_of_the_target}` to scan all ports and identify running services.
- Example**: Port 6379/tcp open, using the Redis service.

## Obtaining a Point of Support

### 1. redis connection
- Redis**: Redis (REmote DIctionary Server) is an advanced open-source NoSQL database used as a database, cache and message broker.
- How it works**: Redis stores data as key-value pairs in RAM for fast access. It also backs up data to disk to guarantee persistence.
- Security**: Redis requires authentication to access stored data. Configuration errors may allow insecure connections.

### 2. Using scripts and command options
- To find out more about the capabilities of a script or command, type its name followed by the `-h` or `--help` switch. For example, `redis-cli --help`.
- To specify the target host for the connection request, use `redis-cli -h {IP_of_the_target}`.

### 3. Connecting to the Redis server
By executing the above command, we connect to the Redis server. Here are some useful commands:
- **info**: Returns information and statistics about the Redis server.
- select {index}** : Selects the logical Redis database by its index.
- **keys *** : Lists all keys present in the selected database.
- **get {key}** : Displays the stored value for a given key.

### 4 Examples of Redis commands
- To connect: `redis-cli -h {IP_of_the_target}`
- To obtain information about the server: `info``.
- Select database: `select 0`.
- To list keys: `keys *`
- To obtain the value of a key: `get {key}`

## System access

### 1. system exploration
- Use the `keys *` command to list the keys in the selected database.
- Use the `get {key}` command to obtain the value of each key. For example, `get flag` to obtain the value of the flag key containing the hash value (flag) required to validate the task on Hack The Box.

### 2. Submitting the flag
- Copy the hash value obtained and paste it on the Starting Point lab page to obtain the machine property.