# Page Source / Acme IT Support – TryHackMe

## Objective
Identify hidden information within a web application using basic enumeration techniques.

---

## Approach

This challenge focused on analyzing a website to discover hidden flags through:

- HTML comments
- Hidden links
- Directory listing
- Framework and changelog analysis

---

## Methodology

### Flag 1 — HTML Comments

The first step was to inspect the page source using:

A hidden HTML comment was found at the top of the page, containing a clue that led to the first flag.

---

### Flag 2 — Hidden Links

By analyzing the page content, a hidden link starting with: `secr`

was discovered, leading to the second flag.

---

### Flag 3 — Directory Listing

Further enumeration revealed external resources.

By exploring directories, it was possible to find a location with **directory listing enabled**, exposing files directly.

Inside this directory, a `flag.txt` file was found.

---

### Flag 4 — Framework Enumeration & Backup Files

The website framework and changelog were analyzed to identify potential weaknesses.

This led to the discovery of an exposed backup file: `/tmp.zip`

which contained the final flag.

---

### Key Findings

- Sensitive information can be exposed through HTML comments
- Hidden links may not be visible in the UI but accessible directly
- Directory listing can reveal critical files
- Backup files left on the server can expose sensitive data
- Framework enumeration can guide further exploitation

---

### Concepts Learned

- View Source analysis
- HTML comments inspection
- Hidden link discovery
- Directory listing exploitation
- Backup file exposure
- Basic web enumeration techniques

---

### Tools Used

- Web browser (view-source)
- Manual enumeration

---

### What I Learned

This challenge reinforced the importance of **thorough enumeration in web applications**.Even without advanced tools, a careful manual analysis can reveal critical vulnerabilities.
