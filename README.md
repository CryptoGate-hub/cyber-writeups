<p align="center">
  <img src="assets/banner.png" alt="cyber-writeups" width="100%">
</p>

<h1 align="center">cyber-writeups</h1>

<p align="center">
  Writeups et notes de progression en cybersécurité — HTB, pwn.college, PortSwigger, CTF.<br>
  Étudiant en Bachelier Sécurité des systèmes · HELMo, Liège · objectif : pentest / sécurité offensive.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Hack%20The%20Box-29%20labs-9FEF00?style=flat-square&logo=hackthebox&logoColor=black" />
  <img src="https://img.shields.io/badge/PortSwigger-29%20labs-orange?style=flat-square&logo=burpsuite&logoColor=white" />
  <img src="https://img.shields.io/badge/pwn.college-128%2F128-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/CTF-3%20challenges-purple?style=flat-square" />
</p>

---

## 📁 Structure du repo

```
cyber-writeups/
├── htb/
│   ├── starting-point/     # 13 machines Tier 1 & 2
│   ├── machines/           # Machines Easy/Medium (Cap, Orion, Nexus, Reactor)
│   ├── sherlocks/          # DFIR investigations (Brutus, Unit42, Campfire-1/2)
│   └── challenges/         # Challenges (Reverse, Web, OSINT, Hardware, Misc)
├── web-security/
│   ├── portswigger/        # 29 labs Web Security Academy
│   └── notes-perso.md      # Fiches : SQLi, XSS, CSRF, SSRF, path traversal
├── linux-luminarium/       # pwn.college — 128/128 labos
├── ctf/                    # Challenges CTF Discord
└── assets/
```

---

## 🗺️ Navigation

### 🟢 Hack The Box

| Section | Contenu | Progression |
|---|---|---|
| [Starting Point](htb/starting-point/README.md) | 13 machines Tier 1 & 2 | ✅ 13/13 |
| [Machines classiques](htb/machines/README.md) | Cap · Orion · Nexus · Reactor | ✅ 4 machines |
| [Sherlocks](htb/sherlocks/README.md) | Brutus · Unit42 · Campfire-1 · Campfire-2 | ✅ 4 Sherlocks |
| [Challenges](htb/challenges/README.md) | Reverse · Web · OSINT · Hardware · Misc | ✅ 8 challenges |

### 🌐 Web Security

| Section | Contenu | Progression |
|---|---|---|
| [PortSwigger](web-security/portswigger/README.md) | SQLi · XSS · CSRF · Clickjacking · SSRF · CORS · File Upload · OS CMDi | ✅ 29 labs |
| [Notes perso](web-security/notes-perso.md) | Fiches de révision web | 📝 En cours |

### 🐧 Linux & Systems

| Section | Contenu | Progression |
|---|---|---|
| [Linux Luminarium](linux-luminarium/README.md) | pwn.college — 15 modules | ✅ 128/128 |

### 🏁 CTF

| Section | Contenu | Progression |
|---|---|---|
| [CTF Discord](ctf/README.md) | Linux · IDOR · Cookie manipulation | ✅ 3 challenges |

---

## 🛠️ Outils utilisés

`nmap` · `Burp Suite` · `Wireshark` · `Metasploit` · `hashcat` · `sqlmap` · `gobuster` · `Python` · `Bash`

---

## 📌 Format d'un writeup

> **Contexte** → **Reconnaissance** → **Exploitation** → **Privilege Escalation** → **Flags** → **Leçons retenues**

Chaque page contient un fil d'Ariane en haut et une navigation précédent/suivant en bas.

---

<p align="center"><i>Repo alimenté au fil des labs et CTF résolus.</i></p>
