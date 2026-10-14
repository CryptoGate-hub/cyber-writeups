[⬅ Retour à Challenges](README.md) · [🏠 Accueil du repo](../../README.md)

# OpenSecret — Web (Very Easy)

| | |
|---|---|
| **Catégorie** | Web |
| **Difficulté** | Very Easy |
| **Récompense** | 100 XP |

## Scénario

> A simple help desk portal where users can submit support tickets. The application uses JWT tokens for session management, but something seems off about how they're implemented. Can you find the security flaw?

## Solution

Le secret JWT est **hardcodé en clair** dans le code source de la page.

1. Ouvre l’instance
2. Clic droit → **View Page Source** (`Ctrl+U`)
3. Cherche `SECRET_KEY` ou `JWT`

Tu trouves directement :

```js
// JWT Secret Key
const SECRET_KEY = "HTB{0p3n_s3cr3ts_ar3_n0t_s3cr3ts}";
```

## Flag

```
HTB{0p3n_s3cr3ts_ar3_n0t_s3cr3ts}
```

## Points clés

- Ne jamais stocker de secrets côté client
- Toujours regarder le source HTML / JS sur les challenges web
- Le nom du challenge (« Open Secret ») était déjà un spoil
