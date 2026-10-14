[🏠 Accueil](../../README.md) · [HTB](../README.md)

# HTB — Machines classiques

Machines Easy/Medium résolues après le Starting Point. Une page par machine, avec diagramme d'attaque, commandes complètes et leçons retenues.

<p align="center">
  <img src="https://img.shields.io/badge/Easy-3-9FEF00?style=flat-square" />
  <img src="https://img.shields.io/badge/Medium-1-FFB347?style=flat-square" />
</p>

---

## Machines résolues

| Machine | OS | Difficulté | Services | Vulnérabilité principale |
|---|---|---|---|---|
| [Cap](cap.md) | Linux | Easy | FTP · SSH · Web (Flask) | IDOR → PCAP credentials + cap_setuid |
| [Orion](orion.md) | Linux | Easy | SSH · HTTP (CraftCMS) | CVE-2025-32432 (RCE) + CVE-2026-24061 |
| [Nexus](nexus.md) | Linux | Easy | SSH · HTTP (Gitea + Krayin CRM) | Credential leak (git history) + File upload RCE |
| [Reactor](reactor.md) | Linux | Medium | SSH · HTTP (Next.js 15) | CVE-2025-55182 (React2Shell RCE) + Node.js `--inspect` privesc |

---

## Leçons clés

- **IDOR** : toujours tester les identifiants numériques incrémentaux dans les URLs
- **PCAP** : analyser les captures réseau avec Wireshark/tshark, filtrer les protocoles non chiffrés
- **Linux capabilities** : `getcap -r /` est aussi important que `find / -perm -4000` pour la privesc
- **Git history** : des credentials supprimés restent accessibles dans l'historique des commits
- **`os.path.join()`** : ne sanitise pas les séquences `..` — vulnérable au path traversal
- **Node.js `--inspect`** : ouvre un débogueur V8 — si exposé en local par root, tunnel SSH → RCE
- **MD5 sans sel** : crack quasi immédiat avec rockyou.txt — chercher les fichiers `.db`
