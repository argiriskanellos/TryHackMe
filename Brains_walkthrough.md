
# Brains Walkthrough

## Port Scan

We start again with a port scan:
```
nmap -sV [Target_Machine_Ip] -T5
```
### Scan Results
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-07 13:16 EEST
Warning: 10.10.35.165 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.10.35.165
Host is up (0.090s latency).
Not shown: 967 closed tcp ports (conn-refused), 30 filtered tcp ports (no-response)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http     Apache httpd 2.4.41 ((Ubuntu))
50000/tcp open  ibm-db2?
```

We observe that on port 50000, a relational database management system is running.  
Navigating to: `http://10.10.35.165:50000`, we see a login form for the TeamCity tool.  
Searching for exploits for version 2023.11.3, we find that the application is vulnerable to Remote Code Execution (RCE).  

We download the exploit from [CVE-2024-27198-RCE](https://github.com/W01fh4cker/CVE-2024-27198-RCE).

After executing the command:
```
pip install requests urllib3 faker
```
to install the required packages, we run the following command:
```
python3 CVE-2024-27198-RCE.py -t http://[Target_Machine_Ip]:50000
```

From there, navigating through the system, we find that in the path `/home/ubuntu`, there is a `flag.txt` file with the flag:
```
THM{f______________________________4}
```

---

## Detection

In the next task, we deal with detection.  
Opening Splunk, we go to **Search your data**.  
From there, we begin to create a search query like this:
```
index=* source="/var/log/auth.log"  ("useradd" OR "adduser" OR "new user" OR "created user")
```

In this query:
- `index=*`: Searches across all indexes available in Splunk.
- `source="/var/log/auth.log"`: Limits the search to logs originating from the `/var/log/auth.log` file. This file contains information about security and authentication events on Linux systems.
- `("useradd" OR "adduser" OR "new user" OR "created user")`: Searches for any of these phrases related to the creation of new users on the system.

Running this query, one of the results is the following:
```
7/4/24 10:32:37.000 PM
Jul  4 22:32:37 brains useradd[11207]: new user: name=e______r, UID=1001, GID=1001, home=/home/e______r, shell=/bin/bash, from=/dev/pts/0

host = brains source = /var/log/auth.log sourcetype = auth_logs
```

From this, we can see the name of the malicious user.

---

Next, we execute the following command:
```
source="/var/log/dpkg.log" "install"
```

`/var/log/dpkg.log`: This log file contains information about the installation, upgrade, and removal actions of packages managed via `dpkg`. Every time a package is installed or removed, that action is logged in this file.

The results we get are quite many, but if we look carefully, we will find the correct answer:
```
7/4/24 10:58:25.000 PM
2024-07-04 22:58:25 install d___________r:amd64 <none> 1.0

host = brains source = /var/log/dpkg.log sourcetype = packages
```

---

Now, let's search for a plugin in TeamCity:
```
source="/opt/teamcity/*" "plugin" "uploaded"
```

This gives us only one result:
```
7/4/24 10:08:31.921 PM
[2024-07-04 22:08:31,921]   INFO - s.buildServer.ACTIVITIES.AUDIT - plugin_uploaded: Plugin "A______Y" was updated by "user with id=11" with comment "Plugin was uploaded to /home/ubuntu/.BuildServer/plugins/A______Y.zip"

host = brains source = /opt/teamcity/TeamCity/logs/teamcity-activities.log sourcetype = teamcity_activities
```
