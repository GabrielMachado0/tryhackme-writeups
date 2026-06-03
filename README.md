# 📓 TryHackMe Writeups

A collection of hands-on penetration testing writeups from my journey through TryHackMe.  
Each writeup documents the full methodology — reconnaissance, enumeration, exploitation, and privilege escalation.

---

## 🎯 Objectives

- Build practical penetration testing skills through real-world labs
- Document methodology and thought process for each machine
- Develop a consistent approach to enumeration and exploitation
- Track progress and lessons learned over time

---

## ✅ Labs Completed

### Easy
| Lab | Category | Topics |
|-----|----------|--------|
| [Startup](./easy/startup/) | Linux / Web / Network | FTP anonymous login, PHP webshell, RCE, pcap analysis, cron privesc |
| [LazyAdmin](./easy/lazyadmin/) | Linux / Web / CMS | SweetRice CMS, MySQL backup exposure, MD5 cracking, sudo misconfiguration |
| [Page Source](./easy/pagesource/) | Web / Enumeration | HTML comments, hidden links, directory listing, backup file exposure |
| [Compiled](./easy/compiled/) | Reverse Engineering | Ghidra, static analysis, strcmp logic, hardcoded passwords |
| [The Game](./easy/thegame/) | Reverse Engineering | strings, grep, plain text secrets in binaries |

---

## 📈 Learning Path Progress

- ✅ Pre Security Path — Completed
- 🔄 Jr Penetration Tester Path — In Progress

---

## 🧠 Methodology

Every writeup follows the same structure:

```
1. Reconnaissance  →  nmap, identify open ports and services
2. Enumeration     →  gobuster, service-specific tools, manual analysis
3. Exploitation    →  gain initial access
4. Privilege Escalation  →  escalate to root/Administrator
5. Flags           →  capture user.txt and root.txt
6. Lessons Learned →  key takeaways from the machine
```

---

## 🛠️ Tools

| Tool | Purpose |
|------|---------|
| `nmap` | Port scanning and service enumeration |
| `gobuster` | Web directory and file enumeration |
| `netcat` | Listeners for reverse shells |
| `Burp Suite` | Web application analysis |
| `Wireshark` | Network traffic analysis |
| `Ghidra` | Binary reverse engineering |
| `John / CrackStation` | Password hash cracking |
| `python3` | Shell upgrades and scripting |
| Linux CLI | General enumeration and exploitation |

---

## 📁 Repository Structure

```
tryhackme-writeups/
└── easy/
    ├── startup/
    │   └── README.md
    ├── lazyadmin/
    │   └── README.md
    ├── pagesource/
    │   └── README.md
    ├── compiled/
    │   └── README.md
    └── thegame/
        └── README.md
```

---

*By [GabrielMachado0](https://github.com/GabrielMachado0) — transitioning into cybersecurity, one lab at a time.*
