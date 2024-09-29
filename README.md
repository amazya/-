# Metasploit-exploit-module-POC
A step-by-step POC of samba server exploit. Scanning with Nmap to find open ports, using Metasploit to find the vulnerability and gain access to the victim.

## ⚠️ Warning
> **This is for educational purpose only!**
> I do not encourage to do it unless you have been granted explicitly the permission to do so. 
  I will not be responsible for any harm caused by using this guide!

## Install 
1. First of all download [Metaspliotable2](https://sourceforge.net/projects/metasploitable2/) Virtual Machine.
2. Extract the folder
3. On VMware choose _open virtual machine_

![Image](https://github.com/amazya/Metasploit-exploit-module-P.O.C/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20032350.png)

## Confirmation of killing 😉
Enter to Metaspliotable2 with the password `msfadmin`

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20032959.png)

Metaspliotable2 will be my victim. Now use `ifconfig` command to discover my target's IP address. I got the IP address to be 192.168.17.130

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20033251.png)

Let's see if my attack machine (-Kali Linux) can ping the victim machine. Use `nmap -sn <victim IP>`.

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20035022.png)

It's clear that my attack machine can reach the victim.
### Scanning open ports
Now I'll find out what ports are open and which services are running in my victim. Use `nmap -sV <victim IP>` to run the service discovery:

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20035539.png)

As you can see it's party time! 🎊🎊 There are to many ports open!
### Look for the treasure (exploits...)
Now that I've performed the service detection step, I know what versions of applications my victim is running. I just have to find out which one of them might be vulnerable. I searched vulnerabilities in my Metasploit database with the `msfconsole` command.

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20040255.png)

Let's find out if the first application in the list, `vsftpd 2.3.4` (which is an ftp service running on port 21) that I found in my service detection phase, has any expliots associated with it. Search for vsftpd in Metaspliot console with `search vsftpd`.

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20040507.png)

Whoa! The first one is already a hit. As you can see, the exploit rank is **excellent** and I can execute backdoor commands with this exploit.

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20040755.png)

I noticed that the `netbios-ssn` service is running on `Samba` in my victim machine's port 139 and 445. There might be an expliot that I could use. But before that, there was no particular version written for the samba application. However, I have an auxiliary module in Metasploit that can find out the version for us. Let's see this in action:
`search smb_version`

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20041523.png)

Now choose the smb scanner:

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20041840.png)

Now let's see the options I have to set up with `show options`:

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20042014.png)

I can set up the RHOSTS and THREADS here. The RHOSTS will be my target and the THREADS determine how fast will the program run. 

Let's set them up: 

`set RHOSTS <victim IP>`

`set THREADS 16`

`show options`

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20042548.png)

Now `run` it.

The output gives me the version of the `Samba - 3.0.20`. Now I can find out the vulnerabilities associated with it. 
Let's try google. 
A simple google search reveals this version is vulnerable to `username map script` command execution.

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20043139.png)

This is also available in Metasploit. Let's perform a search:

`search username map script`

![Image](https://github.com/amazya/Metasploit-exploit-module-POC/blob/main/%D7%A6%D7%99%D7%9C%D7%95%D7%9D%20%D7%9E%D7%A1%D7%9A%202024-04-05%20043743.png)

As you can see, there is an exploit for this vulnerability with an excellent rank. Let's use this one and try to gain access to the metasploitable machine:

`use <exploit number>`

`show options`

You can see that the Payload options are already set up. I will not change it. I only need to `set` up the RHOSTS options.

Now let's `exploit`!

The exploit sets up a reverse TCP handler to accept the incoming connection from the victim machine. Then the exploit completes and opens a session. We can also see that the access level is root. (Use `whoami` command to see it.)
