[⬅ Retour à Starting Point](../README.md) · [🏠 Accueil du repo](../../../README.md)

# Machine 4 — Redeemer

**Redis sans authentification — Port 6379**

| | |
|---|---|
| **Tier** | 1 |
| **OS** | Linux |
| **Service** | Redis (6379) |

## Concept

Montrer qu'une base de données en mémoire exposée sans mot de passe donne un accès direct à toutes les données stockées.

## Méthodologie

### Scan de ports

```bash
nmap -sV -sC 10.129.x.x
```

```
6379/tcp open  redis
```

### Connexion et énumération des clés

```bash
redis-cli -h 10.129.x.x
```

```
127.0.0.1:6379> KEYS *
127.0.0.1:6379> GET flag
```

## Points clés

- Redis sans mot de passe (`requirepass` non défini) expose directement toutes les clés.
- La commande `KEYS *` permet d'énumérer l'intégralité des données stockées.

## Leçon retenue

*Toujours activer `requirepass` sur Redis et restreindre son accès réseau (`bind 127.0.0.1` ou pare-feu strict).*

---

[⬅ Précédent : Dancing](03-dancing.md) · [⬆ Index Starting Point](../README.md) · [Suivant : Appointment ➡](05-appointment.md)
