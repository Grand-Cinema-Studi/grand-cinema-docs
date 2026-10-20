# Décisions d'architecture (ADR)

Un ADR (Architecture Decision Record) est une fiche courte qui consigne **une** décision technique : son contexte, la décision, les alternatives écartées et ses conséquences. Les ADR du front et du back sont tous ici, avec une seule numérotation.

## Index

| N° | Décision | Statut |
|---|---|---|
| [0001](0001-backend-framework-aspnet-core.md) | Framework backend : ASP.NET Core | Accepté |
| [0002](0002-authentication-jwt.md) | Authentification : jeton JWT et mots de passe hachés par PBKDF2 | Accepté |
| [0003](0003-hosting-cloudfront-s3-ec2.md) | Hébergement : front sur S3 + CloudFront, API et base sur un serveur EC2 pour la recette | Accepté |

Décisions déjà prises, à rédiger :

- Clean Architecture et CQRS sans bibliothèque ;
- framework frontend : Angular ;
- blocage des places en base PostgreSQL, sans Redis ;
- stratégie du côté lecture (repository ou projection SQL), avant l'US 5.3.

## Règles

- Un fichier par décision : `NNNN-titre-en-anglais.md`, sur le [modèle](template.md) (format de Michael Nygard). Contenu en français.
- Un ADR **accepté n'est plus modifié**. Si la décision change, on écrit un nouvel ADR qui la remplace ; l'ancien passe au statut « Remplacé par l'ADR NNNN ». L'historique des décisions reste lisible.
- Statuts possibles : Proposé, Accepté, Rejeté, Remplacé par l'ADR NNNN.
- Un ADR est ajouté par une Pull Request (`docs/adr-<sujet>`), citée dans la Pull Request du code qu'il concerne.
