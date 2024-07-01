# Detailed description of the Fawn Write-up process

#FTP
#Protocols
#Reconnaissance
#Anonymous/Guest Access

## Introduction
The "Fawn Write-up" document provides instructions for getting started with Hack The Box, a pentesting platform. It focuses on the "Fawn" machine and includes detailed steps for connecting to a VPN network, scanning ports, and exploiting vulnerable services.

## Introduction to Pentesting

### 1. Enumeration stage
**Objective**: Document the current state of the target to learn as much as possible.
- Use `nmap` to scan open ports and determine which services are running.

## Detailed enumeration

### 1. Ping the target
- Use `ping {IP_of_the_target}` to check the connection. Stop with CTRL+C after four successful responses.

### 2. Scan open ports
- Command**: `nmap -sV {IP_of_the_target}` to identify running services.
- Example**: Port 21/tcp open, using the FTP service.

## Obtaining a Point of Support

### 1. FTP connection
- FTP**: File Transfer Protocol is a network protocol used to transfer files between a client and a server on a computer network.
- How it works**: FTP enables users to upload and download files to and from a server.
- Security**: FTP is not secure, as data, including passwords, is transmitted in clear text. It can be secured with SSL/TLS (FTPS) or replaced by SSH File Transfer Protocol (SFTP).

### 2. Manual brute-force
- FTP connection command**: Use the `ftp {IP_of_the_target}` command to connect to the target's FTP service. If the server allows an anonymous connection, enter `anonymous` as username and any password.
- Login attempts**: As some network devices or hosts may have accounts with default or empty passwords, you can try logging in using common usernames such as `admin`, `administrator`, and `root`.
- **Successful example**: A connection with an anonymous account may succeed if the FTP server is misconfigured.

## System access

### 1. system exploration
- Use the `ls` command to list the files in the current directory.
- Locate the `flag.txt` file containing the hash value (flag) required to validate the Hack The Box task.

### 2. Submitting the Flag
- Read the file with the `cat flag.txt` command.
- Copy the hash value and paste it on the Starting Point lab page to obtain the machine property.