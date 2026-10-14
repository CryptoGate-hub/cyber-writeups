[⬅ Retour à Challenges](README.md) · [🏠 Accueil du repo](../../README.md)

# WebVault Time Machine Investigation — OSINT (Easy)

| | |
|---|---|
| **Catégorie** | OSINT |
| **Difficulté** | Easy |
| **Récompense** | 260 XP |

## Scénario

> Investigate the website **alexmorgan-reviews.net** using the WebVault Internet Archive (style Wayback Machine) to uncover hidden connections and bias against TechCorp products.

## Solution

1. Ouvre l’instance → interface WebVault
2. Sélectionne le site **alexmorgan-reviews.net**
3. Parcours les snapshots (surtout **15 août 2023**)
4. Dans la section **About me** tu trouves :

> Former RivalTech Marketing Specialist

## Flag

```
HTB{Former_RivalTech_Marketing_Specialist}
```

## Points clés

- Les archives web (Wayback / WebVault) révèlent souvent des infos qui ont été retirées
- Toujours regarder les anciennes versions d’un site suspect
