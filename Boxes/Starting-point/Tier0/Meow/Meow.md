# "Meow Write-up"

#Telnet
#Protocols
#Reconnaissance
#Weak Credentials
#Misconfiguration

## Introduction
The "Meow Write-up" document provides instructions for getting started with Hack The Box, a pentesting platform. It focuses on the "Meow" machine and includes detailed steps for connecting to a VPN network, scanning ports, and exploiting vulnerable services.

## Introduction to Pentesting

### 1. Enumeration stage
**Objective**: Document the current state of the target to learn as much as possible.
- Use `nmap` to scan open ports and determine which services are running.

## Detailed enumeration

### 1. Ping the target
- Use `ping {IP_of_the_target}` to check the connection. Stop with CTRL+C after four successful responses.

### 2. Scan open ports
- Command**: `nmap -sV {IP_of_the_target}` to identify running services.
- Example**: Port 23/tcp open, using telnet service.

## Obtaining a Point of Support

### 1. telnet connection
- Telnet**: Telnet (TELetype NETwork) is a network protocol used to provide a command-line interface for remote device management. It was widely used before the adoption of SSH (Secure Shell), which offers encrypted communication.
- How it works**: Telnet lets you connect to a remote host on a network using the TCP/IP protocol. Once connected, you can send commands to the telnet server as if you were physically present on the machine.
- Security**: Telnet is not secure, as data, including passwords, are transmitted in clear text. This makes it vulnerable to interception and man-in-the-middle attacks.

### 2. Manual brute-force
- Telent connection command**: Use the `telnet {IP_of_the_target}` command to connect to the target's telnet service.
- Login attempts**: Since some network devices or hosts may have accounts with default or empty passwords, you can try logging in using common usernames such as `admin`, `administrator`, and `root`.
- **Successful example**: After several attempts, a login with the user `root` without a password may succeed, giving you access to the system.

## System access

### 1. system exploration
- Use the `ls` command to list the files in the current directory.
- Locate the `flag.txt` file containing the hash value (flag) required to validate the Hack The Box task.

### 2. Submitting the Flag
- Read the file with the `cat flag.txt` command.
- Copy the hash value and paste it on the Starting Point lab page to obtain the machine property.