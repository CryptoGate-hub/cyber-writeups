[⬅ Retour à Starting Point](../README.md) · [🏠 Accueil du repo](../../../README.md)

# Machine 1 — MEOW

**Telnet non sécurisé — Port 23**

## Concept

Comprendre pourquoi un protocole non chiffré (Telnet) combiné à une
mauvaise configuration (compte root sans mot de passe) mène à une
compromission totale et immédiate.

## Méthodologie — commandes complètes

> nmap -sV -sC 10.129.x.x
>
> \# → 23/tcp open telnet
>
> telnet 10.129.x.x
>
> \# login: root
>
> \# password: (aucun mot de passe demandé / accès direct)
>
> cat /root/flag.txt

## Points clés

- Telnet transmet identifiants et données en clair, sans aucun
  chiffrement.

- Un compte root accessible sans mot de passe est une faille de
  configuration critique.

- Toujours tester une connexion basique avant de chercher des
  vulnérabilités complexes.

## Leçon retenue

*Désactiver Telnet au profit de SSH, et ne jamais laisser un compte
administrateur sans authentification.*

---

[⬆ Index Starting Point](../README.md) · [Suivant : Fawn ➡](02-fawn.md)
