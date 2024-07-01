# Detailed notes on the "Dancing Write-up" process

#Protocols
#SMB
#Reconnaissance
#Anonymous/Guest Access

## Introduction
The "Dancing Write-up" document provides instructions for getting started with Hack The Box, a pentesting platform. It focuses on the "Dancing" machine and includes detailed steps for connecting to a VPN network, scanning ports, and exploiting vulnerable services.

## Introduction to Pentesting

### 1. Enumeration stage
**Objective**: Document the current state of the target to learn as much as possible.
- Use `nmap` to scan open ports and determine which services are running.

## Detailed enumeration

### 1. Ping the target
- Use `ping {IP_of_the_target}` to check the connection. Stop with CTRL+C after four successful responses.

### 2. Scan open ports
- Command**: `nmap -sV {IP_of_the_target}` to identify running services.
- Example**: Port 445/tcp open, using SMB service.

## Obtaining a Support Point

### 1. SMB connection
- SMB**: Server Message Block is a network protocol used to share files, printers and serial ports between machines on a network.
- How it works**: SMB enables applications to read, create and update files on a remote server. SMB clients must provide a username and password to access SMB shares.
- Security**: SMB requires authentication. However, configuration errors may allow anonymous or guest connections.

### 2. Using scripts and command options
- To find out more about the capabilities of a script or command, type its name followed by the `-h` or `--help` switch. For example, `smbclient -h` or `smbclient --help`.
- To select the target host for the connection request, use `smbclient -L {IP_of_the_target}`.

### 3 Identifying SMB shares
By running the above command, we see that four distinct shares are displayed. Here's what they mean:
- **ADMIN$**: Administrative shares are hidden network shares created by the Windows NT family of operating systems which allow system administrators to remotely access every disk volume on a system connected to the network. These shares cannot be permanently deleted, but can be disabled.
- C$**: Administrative share for the C:\ disk volume. This is where the operating system is hosted.
- IPC$**: Inter-process communication share. Used for inter-process communication via named channels and is not part of the file system.
- WorkShares**: Customized sharing.

## Obtaining a Support Point

### 1. connect to SMB shares
- Try to connect to the various SMB shares identified (`ADMIN$`, `C$`, `IPC$`, `WorkShares`) using the command `smbclient //{IP_of_the_target}/{share_name}`. If the server allows anonymous connection, leave the password field blank and press Enter.
- Successful example** : A connection to the `WorkShares` share may succeed if the SMB server is incorrectly configured.

### 2. System exploration
- Use the `ls` command to list files in the current directory of the connected share.
- Use `cd {directory_name}` to browse directories.
- Locate the `flag.txt` file containing the hash value (flag) required to validate the task on Hack The Box.

### 3. Download the file
- Use the `get {filename}` command to download the desired file. For example, `get flag.txt` to download the flag.txt file to the current directory on your local machine.

### 4. Submitting the flag
- Read the file with the `cat flag.txt` command.
- Copy the hash value and paste it on the Starting Point lab page to obtain the machine property.