[🏠 Accueil](../../README.md) · [HTB](../README.md)

# HTB — Sherlocks

Investigations DFIR (Digital Forensics & Incident Response). Chaque Sherlock fournit des artefacts réels (logs, EVTX, PCAP, Prefetch…) à analyser pour répondre à des questions d'enquête.

> **Disclaimer** — Ces writeups concernent des Sherlocks Very Easy / Easy résolus, publiés à des fins éducatives. Respectez le [Terms of Service](https://www.hackthebox.com/tos) HTB.

---

## Sherlocks résolus

| Sherlock | Difficulté | Artefacts | Technique principale |
|---|---|---|---|
| [Brutus](brutus.md) | Very Easy | `auth.log` · `wtmp` | Brute-force SSH + analyse logs Linux |
| [Unit42](unit42.md) | Very Easy | Sysmon EVTX | Analyse Sysmon — UltraVNC backdoordé |
| [Campfire-1](campfire-1.md) | Very Easy | `SECURITY-DC.evtx` · `Powershell-Operational.evtx` · Prefetch | Kerberoasting — PowerView + Rubeus |
| [Campfire-2](campfire-2.md) | Very Easy | `Security.evtx` | AS-REP Roasting — EID 4768 PreAuthType=0 |

---

## Outils utilisés

`Chainsaw` · `hayabusa` · `evtx_dump` · `Event Viewer` · `grep` · `wtmp` · `last`
