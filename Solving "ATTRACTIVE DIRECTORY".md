# Solving "ATTRACTIVE DIRECTORY": A Step-by-Step Walkthrough of a Vulnerable Machine on TryHackMe

Welcome to this comprehensive writeup detailing the successful exploitation of " [**ATTRACTIVE DIRECTORY**](https://tryhackme.com/room/attacktivedirectory)" a vulnerable machine hosted on [Tryhackme](https://tryhackme.com/dashboard) . In this walkthrough, we will delve into the intricate process of uncovering and exploiting various weaknesses to gain full control of the machine. 
By sharing my thought process and the tools utilized throughout the journey, this writeup aims not only to document my achievements but also to demonstrate the valuable insights and skills I gained in the process.
Let's begin the journey of unraveling the secrets hidden within "ATTRACTIVE DIRECTORY"!

## Task 1 (Deploy the machine)
A simple step to access the Virtual Machine, you will need to first connect to our network using OpenVPN. By going to your [access](https://tryhackme.com/access) page. Select your VPN server of choice and download your configuration file. 

![image](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/6e5d9f26-9caf-4520-bf92-6dbabd6dfda4)

After downloading the configuration file we connect to the VPN by using this command

```$ sudo openvpn /home/kali/Desktop/coedoverheated41.ovpn```

![2023-07-26 10_57_18-Window](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/14aadfe9-48d0-4613-ac53-86ac23a49085)

After connecting to this VPN leave this tap opened and open a new tap to start solving the machine

# Task 2 (Setup)
In this step, he wants you to install some helping tools like **impacket**, **Bloodhound**, and **Neo4j**
By simply running these commands

> ```$ git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket```
>> ```$ pip3 install -r /opt/impacket/requirements.txt```
>>> ```$ cd /opt/impacket/ && python3 ./setup.py install```

IF YOU ARE USING ON ATTACKBOX U ARE READY TO GO !

# Task 3 (Welcome to Attacktive Directory)
To begin our reconnaissance process, we kick off with a fundamental step called "basic enumeration," which starts with a Nmap scan. Nmap, a sophisticated and well-honed utility, plays a crucial role in identifying open ports, running services, and even determining the underlying operating system on the target device. Nevertheless, it's worth mentioning that Nmap's capabilities, despite being comprehensive, may not always lead to complete enumeration of all services.

As we delve deeper into the challenge, we must recognize that relying solely on Nmap might not provide a comprehensive picture. Thus, to ensure a more thorough enumeration, we will complement our initial Nmap scan with other specialized utilities designed to scrutinize the services running on the target device. By adopting this holistic approach, we can effectively uncover additional information and maximize our enumeration potential.

To run the **nmap scann** we will use this command
```$ sudo nmap -sV -sC -oN nmap.out 10.10.76.108```

![2023-07-26 11_10_59-Practical Ethical Hacking ctb - F__PROGRAM FILES(DONT TOUCH)_CherryTree - Cherry](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/3968b60f-aa8b-4ab2-a0be-76d4c2c1dd3b)

Upon conducting our nmap enumeration, we have discovered a multitude of services running on the target machine, each playing a unique role in its setup. The identified services include DNS, IIS (Internet Information Services), Kerberos, RPC (Remote Procedure Call), NetBIOS, and the prominent Active Directory among others

**Questions:**
> **What tool will allow us to enumerate port 139/445?**  `enum4linux`
>> because Ports 139 and 445 are used by SMB. To enumerate SMB a great tool to use is enum4linux.

> **What is the NetBIOS-Domain Name of the machine?**  `THM-AD`
>>By using **enum4linux** We found the NetBIOS-Domain Name mentioned many times

> **What invalid TLD do people commonly use for their Active Directory Domain? (TLD means top level domain)**  `.local`
>> In our Nmap results, we spotted an Active Directory (AD) Domain named "spookysec.local" on port 3389.

## Task 4 (Enumerating Users via Kerberos)
In our exploration, we observed numerous active ports, one of which stood out as `Kerberos`, a crucial authentication service within Active Directory. Leveraging [Kerbrute](https://github.com/ropnop/kerbrute/releases) , we can conduct brute force attempts to discover users, passwords, and even attempt password spraying.

However, for this particular machine, we'll utilize a customized [User List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt) and [Password List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt) to expedite the enumeration process and password hash cracking. It's important to note that brute forcing credentials may **NOT** be advisable due to the risk of triggering account lockout policies, which are beyond our ability to enumerate on the domain controller. As such, we'll proceed with caution, ensuring our approach aligns with the limitations of the challenge environment.
