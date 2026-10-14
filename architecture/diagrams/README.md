# Schémas

Les schémas du projet sont écrits en **Mermaid**, directement dans les documents Markdown : GitHub les affiche, et ils se modifient et se relisent en Pull Request comme du texte. Un schéma vit dans le document qu'il illustre ; ce dossier les recense et accueille leurs exports en image.

## Index

| Schéma | Type | Où |
|---|---|---|
| Composants de l'application (front, API, base, Stripe) | Mermaid | [overview.md](../overview.md#composants) |
| Couches du backend (Clean Architecture) | Mermaid | [overview.md](../overview.md#architecture-du-backend) |
| Architecture de production (serveur, conteneurs, proxy) | texte | [environments.md](../../deployment/environments.md#production--architecture-retenue) |
| Modèle de données (7 tables) | Mermaid | [erd.md](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/blob/dev/docs/database/erd.md), dans le dépôt backend, à côté des migrations |
| Parcours d'une requête (US 1.1) | texte | [from_scratch.md](https://github.com/Grand-Cinema-Studi/grand-cinema-backend/blob/dev/from_scratch.md), section « Référence – Parcours d'une requête dans le code » |

## Exports en image

Les copies en PDF ne savent pas afficher Mermaid : un schéma utilisé dans une copie est exporté en PNG dans ce dossier, sous le nom de son document d'origine (`overview-components.png`, …). L'export se fait depuis l'aperçu Mermaid de l'éditeur ou avec l'outil en ligne de commande de Mermaid :

```bash
npx -p @mermaid-js/mermaid-cli mmdc -i schema.mmd -o schema.png
```

La source fait foi : on modifie le schéma dans le document, puis on refait l'export.
