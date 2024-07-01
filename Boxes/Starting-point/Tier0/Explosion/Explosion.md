# Writeup Explosion

#Programming
#RDP
#Reconnaissance
#Weak Credentials

## Introduction

In a corporate environment, employees sometimes need support from their peers. This scenario assumes a new employee has access to technical tools on their corporate Windows OS to do their job effectively. Among these tools are remote access software that allows them to connect to other hosts running either Linux or Windows to support their teammates or customers. In this case, we will explore the risks involved with misconfigurations in remote access tools.

**RDP** Remote desktop protocol (RDP) is a secure network communications protocol developed by Microsoft. It enables network administrators to remotely diagnose problems that individual users encounter and gives users remote access to their physical work desktop computers.

**xfreerdp** is a command-line client that is part of the FreeRDP project, which provides a free implementation of the Remote Desktop Protocol (RDP). RDP is a proprietary protocol developed by Microsoft that allows users to connect to another computer over a network and interact with its desktop interface as if they were physically present at the machine.

## Enumeration

- `nmap -sV -sC {TARGET_ip}`
    - 135/tcp - Microsoft Windows RPC
    - 139/tcp - netbios-ssn
    - 445/tcp - microsoft-ds
    - 3389/tcp - Microsoft Terminal Services (RDP)

## Connect RDP with xfreerdp

You could try common username conventions, such as:

Administrator
Guest
The NetBIOS name: EXPLOSION

- `xfreerdp /v:{TARGET_IP}:3390 /u:administartor`

flag : `951fa96d7830c451b536be5a6be008a0`