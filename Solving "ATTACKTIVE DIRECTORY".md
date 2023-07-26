# Solving "ATTACKTIVE DIRECTORY": A Step-by-Step Walkthrough of a Vulnerable Machine on TryHackMe

Welcome to this comprehensive writeup detailing the successful exploitation of " [**ATTACKTIVE DIRECTORY**](https://tryhackme.com/room/attacktivedirectory)" a vulnerable machine hosted on [Tryhackme](https://tryhackme.com/dashboard) . In this walkthrough, we will delve into the intricate process of uncovering and exploiting various weaknesses to gain full control of the machine. 
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

By running this command
```./kerbrute_linux_amd64 usernum -d spookysec.local --dc 10.10.76.108 ~/attactive_directory/usernames.txt ```

![2023-07-26 12_06_06-Practical Ethical Hacking ctb - F__PROGRAM FILES(DONT TOUCH)_CherryTree - Cherry](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/691b6a72-a2df-4386-aa08-5b62db79af5d)

There are some interesting accounts here. But surprisingly we found that svc-admin has no pre-auth (TGT) that must be required to get the TGC
Not only that but also it provides us with the **Kerberos hash** 

```$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:408ee4a3e91ec877b931d35c56364c77$63dc9e093d6f3ddfd0074033786ed4d4d6e5f3e9f27be7f98866c0c91c4271c6c8a721eafa9e343a2b9638da64fe71d7563c31e51e6aac0686ba9025ab8ff2d41b8b24f38888cd803c70568744a12daa95cca16b73fa6bc5b20f1fb697b29fd1fe39fa0553ae07ad7e6e2f5232e306ee2abf3ee2ba8ebc704bc96f0d60cd245f96f4caa7c20c3a673fba2b25a384593b01e334560348a146d9168e1fc594b8c59e11382193bd2b3f1c421f9d5fdc61167c8f3bfa18d60fc6fca79923c16b707927719330363b593c28ccc0c7dd2c5e7696b43d45a4bc016341f773805c53f51d2b6ae4a0fa3c3280a18a9d53d9b5fd08337c```

By Looking-up the hash head in **hashcat help** we knew that the type of the hash is ```18200 kerberos 5, etype 23, AS-REP```
![2023-07-26 12_17_35-Window](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/98bf732f-e24f-45a2-b2e0-ca91c6110a64)

***Cracking The Hash***

By using **HashCat** we can try to Decrypt the hash by using the [Password List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt) provided 

By using this command ``` hashcat -m 18200 TGT.txt pass.txt -o out.txt```
![2023-07-26 12_21_01-Window](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/3236811c-cfad-4513-9747-6e6697235f7c)

**THE OUTPUT**
![2023-07-26 12_22_57-Window](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/4783e424-9658-4b24-9f80-5d474200ef1e)

***SO THE PASSWORD AFTER CRACKING IS : management2005***

**Questions:**
> What command within Kerbrute will allow us to enumerate valid usernames?  `userenum`

> What notable account is discovered? (These should jump out at you)  `svc-admin`

> What is the other notable account is discovered? (These should jump out at you)  `backup`


# Task 6 (Back to the Basics)
Having obtained the user's account credentials, our access within the domain has substantially increased. With this elevated privilege, we can proceed to enumerate any shares that the domain controller might be granting to users. This step allows us to gain valuable insights into the network's file-sharing architecture and potentially uncover sensitive information or resources.

We can do this by using **smbclient tool**
By using this command ```smbclient -L \\\\10.10.148.99\\ -U svc-admin```

![2023-07-26 12_27_06-Window](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/cbbeed0d-24e5-4a82-a0fd-f9f0e8a8f45b)

We Found that **backup** is an interesting Folder 
so by Grapping it ```smbclient \\\\10.10.148.99\\backup -U svc-admin ```
We found another interesting File **backup_credentials.txt** 
Download it by using the ```get <file name>``` command 

We Found that the credentials are in **base64** so by Decoding it 
![2023-07-26 12_33_22-Window](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/295d618c-8601-4f4d-ab30-4f5c731e55e0)

**We got the Result is : backup@spookysec.local:backup2517860**

**Questions:**
> Using utility can we map remote SMB shares?  `smbclient`

> Which option will list shares?  `-L`

> How many remote shares is the server listing?  `6`

> There is one particular share that we have access to that contains a text file. Which share is it?  `backup`

> What is the content of the file? `YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw`

> Decoding the contents of the file, what is the full contents?  `backup@spookysec.local:backup2517860`

# Task 7 (Elevating Privileges within the Domain)
Having acquired new user account credentials, our privileges on the system have been significantly elevated. Notably, we came across a user account named "backup," which sparked our curiosity about its purpose

As this is the **backup account** for the **Domain Controller**. This account has a unique permission that allows all Active Directory changes to be synced with this user account. This includes password hashes.

So We’ll use one of the ```impacket tools``` called ```secretsdump.py``` to dump password hashes.

**THE RESULT IS**
![2023-07-26 12_41_30-Window](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/bdc298db-af90-459e-9cf6-82f70dc8637e)

As shown we have successfully gained the Administrator credentials 

**Questions:**
> What method allowed us to dump NTDS.DIT?  `DRSUAPI`

> What is the Administrators NTLM hash?  `0e0363213e37b94221497260b0bcb4fc`

> What method of attack could allow us to authenticate as the user without the password?  `Pass the hash`

> Using a tool called Evil-WinRM what option will allow us to use a hash?  `-H`

# Task 8 (Flag Submission Panel)
Submit the flags for each user account. Which are located on each user's desktop

**Firstly** we use must login as an Administrator which will be done by passing Administrator account’s hash to log in by passing the hash method via ```evilwinrm``` tool, we’ll be Administrator user on the system.

``` evil-winrm -i <target ip> -u Administrator -H 0e0363213e37b94221497260b0bcb4fc ```
**OR** there is another method by using ```psexex.py``` from ```impact tools``` 

![2023-07-26 12_51_43-Window](https://github.com/zeyadtarekXx/Attractive-Directory-Writeup/assets/61972622/0de1722d-6ec7-4e91-8c32-de3157524d88)

BUT ether way u will eventually reach the same goal

Then **FINALLY** by navigating through the Desktops of the users we will capture the Flags and the MAchine will be solved !!

**FLAGS**

**svc-admin**
> **_Answer_**: _TryHackMe{K3rb3r0s_Pr3_4uth}_

**backup**
> **_Answer_**:  _TryHackMe{B4ckM3UpSc0tty!}_

**Administrator**
> **_Answer_**:  _TryHackMe{4ctiveD1rectoryM4st3r}_

THANK YOU FOR READING
