[⬅ Retour à Starting Point](../README.md) · [🏠 Accueil du repo](../../../README.md)

# Machine 2 — Fawn

**FTP anonyme — Port 21**

| | |
|---|---|
| **Tier** | 1 |
| **OS** | Linux |
| **Service** | FTP (21, vsftpd) |

## Concept

Illustrer le risque d'un service FTP autorisant les connexions anonymes en lecture.

## Méthodologie

### Scan de ports

```bash
nmap -sV -sC 10.129.x.x
```

```
21/tcp open  ftp  vsftpd
Anonymous FTP login allowed
```

### Connexion anonyme et récupération du flag

```bash
ftp 10.129.x.x
```

```
Name: anonymous
Password: (vide, Entrée)
```

```bash
ls
get flag.txt
exit
cat flag.txt
```

## Points clés

- Le FTP anonyme permet un accès en lecture sans authentification réelle.
- Toujours tester `anonymous` comme identifiant en premier sur un service FTP.

## Leçon retenue

*Désactiver l'accès anonyme sur les serveurs FTP de production, ou le restreindre à un répertoire public strictement contrôlé.*

---

[⬅ Précédent : Meow](01-meow.md) · [⬆ Index Starting Point](../README.md) · [Suivant : Dancing ➡](03-dancing.md)
