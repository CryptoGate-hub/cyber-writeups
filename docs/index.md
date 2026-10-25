<p align="center">
  <img src="assets/banner.png" alt="cyber-writeups" width="100%">
</p>

<h1 align="center">cyber-writeups</h1>

<p align="center">
  Writeups et notes de progression en cybersécurité — HTB, pwn.college, PortSwigger, CTF.<br>
  Étudiant en Bachelier Sécurité des systèmes · HELMo, Liège · objectif : pentest / sécurité offensive.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/HTB%20Starting%20Point-13%2F13-9FEF00?style=flat-square&logo=hackthebox&logoColor=black" />
  <img src="https://img.shields.io/badge/HTB%20Machines-4-9FEF00?style=flat-square&logo=hackthebox&logoColor=black" />
  <img src="https://img.shields.io/badge/HTB%20Sherlocks-4-9FEF00?style=flat-square&logo=hackthebox&logoColor=black" />
  <img src="https://img.shields.io/badge/HTB%20Challenges-8-9FEF00?style=flat-square&logo=hackthebox&logoColor=black" />
  <br>
  <img src="https://img.shields.io/badge/PortSwigger-29%20labs-orange?style=flat-square&logo=burpsuite&logoColor=white" />
  <img src="https://img.shields.io/badge/pwn.college-128%2F128-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/CTF-3%20challenges-purple?style=flat-square" />
</p>

---

## ⚡ Writeups phares

Un aperçu rapide des writeups les plus complets du repo, pour qui n'a que deux minutes :

| Writeup | Pourquoi le lire |
|---|---|
| [🟢 Reactor](htb/machines/reactor.md) | RCE sur une CVE React/Next.js 2025 + privesc via débogueur Node.js `--inspect` |
| [🟢 Nexus](htb/machines/nexus.md) | Fuite de creds dans l'historique Gitea + RCE CRM + directory traversal fait main (objets git forgés) |
| [🔎 Campfire-1](htb/sherlocks/campfire-1.md) | Investigation DFIR complète — Kerberoasting détecté via EVTX Sysmon/PowerShell |
| [🚀 Unified](htb/starting-point/labs/13-unified.md) | Log4Shell de bout en bout, du JNDI à la privesc MongoDB |
| [🔴 Labo AD — Kerberoasting](labo/ad-kerberoasting.md) | Labo construit de A à Z : AD, Kerberoasting avec Impacket, détection Wazuh SIEM custom |

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
├── labo/
│   └── ad-kerberoasting/   # Labo AD + Kerberoasting + Wazuh
└── assets/
```

---

## 🗺️ Navigation

### 🟢 Hack The Box

| Section | Contenu | Progression |
|---|---|---|
| [Starting Point](htb/starting-point/) | 13 machines Tier 1 & 2 | ✅ 13/13 |
| [Machines classiques](htb/machines/) | Cap · Orion · Nexus · Reactor | ✅ 4 machines |
| [Sherlocks](htb/sherlocks/) | Brutus · Unit42 · Campfire-1 · Campfire-2 | ✅ 4 Sherlocks |
| [Challenges](htb/challenges/) | Reverse · Web · OSINT · Hardware · Misc | ✅ 8 challenges |

### 🌐 Web Security

| Section | Contenu | Progression |
|---|---|---|
| [PortSwigger](web-security/portswigger/) | SQLi · XSS · CSRF · Clickjacking · SSRF · CORS · File Upload · OS CMDi | ✅ 29 labs |
| [Notes perso](web-security/notes-perso.md) | Fiches de révision web | 📝 En cours |

### 🐧 Linux & Systems

| Section | Contenu | Progression |
|---|---|---|
| [Linux Luminarium](linux-luminarium/) | pwn.college — 15 modules | ✅ 128/128 |

### 🏁 CTF

| Section | Contenu | Progression |
|---|---|---|
| [CTF Discord](ctf/) | Linux · IDOR · Cookie manipulation | ✅ 3 challenges |

### 🏗️ Labo Personnel

| Section | Contenu | Progression |
|---------|---------|-------------|
| [Active Directory](labo/ad-kerberoasting.md) | Kerberoasting · Détection Wazuh · Règle SIEM custom | ✅ 1 projet |

---

## 🛠️ Outils utilisés

`nmap` · `Burp Suite` · `Wireshark` · `Metasploit` · `hashcat` · `sqlmap` · `gobuster` · `impacket` · `evil-winrm` · `Chainsaw` · `Python` · `Bash`

---

## 📌 Format d'un writeup

> **Contexte** → **Reconnaissance** → **Exploitation** → **Privilege Escalation** → **Flags** → **Leçons retenues**

Chaque page suit la même structure qu'un writeup officiel HTB : schéma de la chaîne d'attaque (Mermaid), commandes complètes en blocs de code, tableau récapitulatif des flags/réponses, et une section "Points clés" qui capitalise les leçons transférables à d'autres machines. Chaque page contient un fil d'Ariane en haut et une navigation précédent/suivant en bas.

### Légende des difficultés

| Badge | Signification |
|---|---|
| 🟩 Very Easy / Easy | Prise en main, une seule vulnérabilité ou une chaîne courte |
| 🟧 Medium | Chaîne d'exploitation à plusieurs étapes, CVE récente ou logique applicative non triviale |
| 🟥 Hard+ | Pas encore abordé dans ce repo |

---

<p align="center"><i>Repo alimenté au fil des labs et CTF résolus. Toutes les machines HTB publiées ici sont Retired ou issues d'événements terminés — writeups à but strictement éducatif.</i></p>

