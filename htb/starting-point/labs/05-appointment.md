[⬅ Retour à Starting Point](../README.md) · [🏠 Accueil du repo](../../../README.md)

# Machine 5 — APPOINTMENT

**Injection SQL sur formulaire web — Port 80**

## Concept

Contourner une authentification web via une injection SQL classique dans
le champ nom d'utilisateur.

## Méthodologie — commandes complètes

> nmap -sV -sC 10.129.x.x
>
> \# → 80/tcp http (formulaire de prise de rendez-vous / login)
>
> \# Dans le champ "username" du formulaire :
>
> admin' --
>
> \# Champ "password" : n'importe quelle valeur
>
> \# → authentification contournée, accès à la page confirmant le flag

## Points clés

- Une requête SQL construite par concaténation directe (sans requêtes
  préparées) est vulnérable à l'injection.

- admin' -- commente le reste de la clause WHERE, validant la connexion
  sans connaître le vrai mot de passe.

## Leçon retenue

*Toujours utiliser des requêtes préparées (prepared statements /
requêtes paramétrées) pour empêcher l'injection SQL.*

---

[⬅ Précédent : Redeemer](04-redeemer.md) · [⬆ Index Starting Point](../README.md) · [Suivant : Sequel ➡](06-sequel.md)
