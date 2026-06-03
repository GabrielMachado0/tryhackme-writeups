# Page Source / Acme IT Support — TryHackMe

**Difficulty:** Easy  
**Category:** Web / Enumeration  
**Date:** 03/06/2026  
**Status:** ✅ Completed

---

## 📋 Table of Contents

- [Reconnaissance](#reconnaissance)
- [Enumeration](#enumeration)
- [Exploitation](#exploitation)
- [Flags](#flags)
- [Lessons Learned](#lessons-learned)

---

## 🔍 Reconnaissance

> Goal: identify the attack surface of the machine.

**Initial observations:**
- Static web application from Acme IT Support
- No login panel or dynamic functionality visible at first glance
- Target for manual enumeration rather than active scanning

---

## 📡 Enumeration

> Goal: extract as much information as possible from the discovered services.

### Flag 1 — HTML Source Inspection

Inspected the raw HTML of the page using the browser:

```
view-source:http://<THM_IP_MACHINE>
```

A hidden HTML comment was found near the top of the source containing the first flag.

---

### Flag 2 — Hidden Links

Further analysis of the page source revealed a hidden anchor link not visible in the rendered UI, starting with:

```
secr...
```

Navigating to the full URL directly in the browser revealed the second flag.

---

### Flag 3 — Directory Listing

External resources referenced in the page source were explored manually. One directory had **directory listing enabled**, exposing its contents without authentication.

Inside the exposed directory, a `flag.txt` file was found and accessed directly.

---

### Flag 4 — Framework Enumeration & Backup File

The website framework and its changelog were analysed to identify the version in use. This led to the discovery of an exposed backup file accessible at:

```
/tmp.zip
```

Downloading and extracting the archive revealed the final flag.

---

## 💥 Exploitation

> Goal: gain initial access to the machine.

**Vulnerability found:** Information disclosure through HTML comments, hidden links, directory listing, and exposed backup files.

**Tools used:** Browser (view-source), manual enumeration

No automated tools were required for this challenge. All flags were discovered through careful manual analysis of the web application's source code and directory structure.

---

## 🚩 Flags

| Flag | Location | How Found |
|------|----------|-----------|
| Flag 1 | HTML comment in page source | view-source inspection |
| Flag 2 | Hidden link (`secr...`) | Manual source analysis |
| Flag 3 | `flag.txt` in exposed directory | Directory listing |
| Flag 4 | Inside `/tmp.zip` | Framework + backup file enumeration |

---

## 📚 Lessons Learned

1. **HTML comments can leak sensitive information** — always inspect page source before reaching for automated tools
2. **Hidden links exist outside the visible UI** — full source analysis often reveals paths not rendered on screen
3. **Directory listing exposes files directly** — misconfigured web servers can make entire directories browsable
4. **Backup files left on the server are a critical risk** — framework version enumeration can reveal predictable backup file locations

---

## 🛠️ Tools Used

- Browser (`view-source`)
- Manual enumeration

---

*Writeup by GabrielMachado0*
