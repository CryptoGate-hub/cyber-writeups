[🏠 Accueil du repo](../../README.md) · [HTB](../)

# Hack The Box — Machines classiques

Machines Easy/Medium résolues après le Starting Point. Une page par machine, avec diagramme d'attaque, commandes complètes et leçons retenues.

## Résumé des machines

Vue d'ensemble des machines classiques HTB résolues (hors Starting
Point), avec le service et la vulnérabilité principale exploitée pour
chacune.

|             |                                             |                    |                                                 |
|-------------|---------------------------------------------|--------------------|-------------------------------------------------|
| **Machine** | **Service(s)**                              | **Port(s)**        | **Vulnérabilité principale**                    |
| [Cap](cap.md)         | FTP + SSH + Web (Flask)                     | 21, 22, 80         | IDOR + capability cap_setuid                    |
| [Orion](orion.md)       | SSH + HTTP (CraftCMS 5.6.16) + Telnet local | 22, 80 (+23 local) | CVE-2025-32432 (RCE) + CVE-2026-24061 (telnetd) |


## Conclusion générale

### Ce que ces machines enseignent

- Toujours tester les IDOR sur des identifiants numériques incrémentaux
  exposés dans une URL.

- Analyser systématiquement tout fichier de capture réseau (pcap)
  récupéré avec Wireshark/tshark, en filtrant sur les protocoles non
  chiffrés.

- getcap -r / est aussi important que find / -perm -4000 pour
  l'énumération de privesc Linux moderne.
