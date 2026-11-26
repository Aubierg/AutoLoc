# Notes de sécurité — AutoLoc

## Règles de routes

| Route | Règle |
|---|---|
| `/api/public/**` | Accessible à tous (`permitAll`) |
| `/api/admin/**` | Réservée au rôle ADMIN |
| `/api/**` | Réservée aux rôles USER et ADMIN |
| Toute autre route | Authentification obligatoire |

La règle `/api/admin/**` est déclarée avant `/api/**` : Spring Security applique la première règle qui correspond. Sinon, un simple USER pourrait accéder aux routes admin.

## Authentification et autorisation

- Authentification : vérifier qui est l'utilisateur (identifiant et mot de passe).
- Autorisation : vérifier ce que cet utilisateur a le droit de faire (son rôle).

## 401 et 403

- 401 Unauthorized : l'utilisateur n'est pas authentifié.
- 403 Forbidden : l'utilisateur est authentifié, mais son rôle ne l'autorise pas.

## Encodage des mots de passe

Les mots de passe sont encodés avec BCrypt : un hachage à sens unique, avec sel. Si la base est volée, les mots de passe ne peuvent pas être lus directement.

## Limites des utilisateurs en mémoire

- Adaptés à un exercice local, pas à la production.
- Impossible de créer ou modifier un utilisateur sans redéployer.
- Les mots de passe d'exemple sont visibles dans le code source.

## Charger les utilisateurs depuis MySQL

- Une entité Utilisateur (identifiant, mot de passe encodé, rôle) et un repository.
- Une implémentation de UserDetailsService qui lit la base.
- L'encodage BCrypt à l'enregistrement.