# Metasploit-exploit-module-POC
A step-by-step POC of samba server exploit. Scanning with Nmap to find open ports, using Metasploit to find the vulnerability and gain access to the victim.
## Install 
1. First of all download [Metaspliotable2](https://sourceforge.net/projects/metasploitable2/) Virtual Machine.
2. Extract the folder
3. On VMware choose _open virtual machine_

![Image](https://github.com/amazya/Metasploit-exploit-module-P.O.C/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20032350.png)

## Confirmation of killing 😉
Enter to Metaspliotable2 with the password `msfadmin`

![Image](https://github.com/user-attachments/assets/843dad1a-bd6f-449f-b8a7-73ba0d242d21)

Metaspliotable2 will be my victim. Now use `ifconfig` command to discover my target's IP address. I got the IP address to be 192.168.17.130

![Image](https://github.com/user-attachments/assets/81ccfc88-7811-4e6a-93d6-b04bca426090)

Let's see if my attack machine (-Kali Linux) can ping the victim machine. Use `nmap -sn <victim IP>`.

![Image](https://github.com/user-attachments/assets/acd070ea-7caa-441f-b5bd-8e699d3bf326)

It's clear that my attack machine can reach the victim.
### Scanning open ports
Now I'll find out what ports are open and which services are running in my victim. Use `nmap -sV <victim IP>` to run the service discovery:

![Image](https://github.com/user-attachments/assets/29dff5d7-0ce3-4a76-b7c0-086acef7e699)

As you can see it's party time! 🎊🎊 There are to many ports open!
### Look for the treasure (exploits...)
Now that I've performed the service detection step, I know what versions of applications my victim is running. I just have to find out which one of them might be vulnerable. I searched vulnerabilities in my Metasploit database with the `msfconsole` command.

![Image](https://github.com/user-attachments/assets/fb14a62b-30dc-4e00-9139-5823af5fa2e2)

Let's find out if the first application in the list, `vsftpd 2.3.4` (which is an ftp service running on port 21) that I found in my service detection phase, has any expliots associated with it. Search for vsftpd in Metaspliot console with `search vsftpd`.

![Image](https://github.com/user-attachments/assets/15e68715-a887-47c7-9168-b2918b587f93)

Whoa! The first one is already a hit. As you can see, the exploit rank is **excellent** and I can execute backdoor commands with this exploit.

![Image](https://github.com/user-attachments/assets/4c79da27-d926-4b66-852e-fa8bb2684b5a)

I noticed that the `netbios-ssn` service is running on `Samba` in my victim machine's port 139 and 445. There might be an expliot that I could use. But before that, there was no particular version written for the samba application. However, I have an auxiliary module in Metasploit that can find out the version for us. Let's see this in action:
`search smb_version`

![Image](https://github.com/user-attachments/assets/f239a533-a2cf-4835-aed6-2cbbf83a3b39)

Now choose the smb scanner:

![Image](https://github.com/user-attachments/assets/7d54254d-fa07-4ff1-ad26-138182a6cac9)

Now let's see the options I have to set up with `show options`:

![Image](https://github.com/user-attachments/assets/d14384c0-a326-4a6f-9c53-ce2ac03bb67b)

I can set up the RHOSTS and THREADS here. The RHOSTS will be my target and the THREADS determine how fast will the program run. 
Let's set them up: 
`set RHOSTS <victim IP>`
`set THREADS 16`
`show options`

![Image](https://github.com/user-attachments/assets/38e2f44a-caf5-453d-86c9-62c4a8ee0392)

Now `run` it.
The output gives me the version of the `Samba - 3.0.20`. Now I can fnd out the vulnerabilities associated with it. Let's try google. A simple google search reveals this version is vulnerable to `username map script` command execution.

![Image](https://github.com/user-attachments/assets/d17d0cf1-a71d-4d60-b6e0-b9dbd04534c3)

This is also available in Metasploit. Let's perform a search:
`search username map script`

![Image](https://github.com/user-attachments/assets/2f8e84b0-689a-4276-bb1e-653e7b234b00)

As you can see, there is an exploit for this vulnerability with an excellent rank. Lets use this one and try to gain access to the metasploitable machine:
`use <exploit number>`
`show options`
You can see that the Payload options are already set up. I will not change it. I only need to `set` up the RHOSTS options.
Now let's `exploit`!
The exploit sets up a reverse TCP handler to accept the incoming connection from the victim machine. Then the exploit completes and opens a session. We can also see that the access level is root. (Use `whoami` command to see it.)
