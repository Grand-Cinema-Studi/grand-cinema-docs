# Poste de développement

Comment préparer un poste et lancer l'application complète (base, API, front) en local. Le détail de chaque composant est dans le `README.md` de son dépôt : ce document donne la vue d'ensemble et l'ordre des étapes.

Les éléments marqués _(à venir)_ sont décidés mais pas encore réalisés.

## Outils

| Outil | Version | Pour |
|---|---|---|
| Git | récente | les trois dépôts |
| SDK .NET | 10, version exacte fixée par `global.json` du backend | compiler, tester et lancer l'API |
| Docker (Docker Desktop sous Windows et macOS) | récente | PostgreSQL en local, tests d'intégration (Testcontainers) |
| Git Bash (sous Windows) | fourni avec Git | les scripts `.sh` et les commandes de la documentation, écrites pour bash |
| Node.js et Angular CLI | versions fixées dans le dépôt frontend _(à venir)_ | compiler et lancer le front |
| Éditeur | VS Code avec C# Dev Kit, ou Visual Studio / Rider | au choix |

Le projet est développé sur deux postes (Windows et Linux) : les versions figées dans les dépôts (`global.json`, `dotnet-tools.json`, image `postgres:16`) garantissent le même résultat sur les deux.

## Récupérer les dépôts

Les trois dépôts dans un même dossier :

```bash
git clone https://github.com/Grand-Cinema-Studi/grand-cinema-backend.git
git clone https://github.com/Grand-Cinema-Studi/grand-cinema-frontend.git
git clone https://github.com/Grand-Cinema-Studi/grand-cinema-docs.git
```

## Lancer l'application

| Ordre | Composant | Commande principale | Adresse | Détail |
|---|---|---|---|---|
| 1 | PostgreSQL | `docker compose up -d` (dans le backend) | `localhost:5432` | [README du backend](https://github.com/Grand-Cinema-Studi/grand-cinema-backend#lancer-le-projet-en-local), étape 2 |
| 2 | Schéma et données de démonstration | `dotnet ef database update …` puis `bash scripts/reset-demo-screenings.sh` | — | README du backend, étape 4 |
| 3 | API | `dotnet run --project src/GrandCinema.Api` | http://localhost:5073 (Swagger : `/swagger`) | README du backend, étape 5 |
| 4 | Front _(à venir)_ | `npm install` puis `npm start` | http://localhost:4200 | README du frontend |

Le front appelle l'API par des chemins relatifs (`/api/...`). En local, le serveur de développement d'Angular redirige ces chemins vers l'API (proxy) _(à venir)_ ; en production, c'est le proxy Caddy qui le fait (voir [environnements](../deployment/environments.md)). Le front n'a donc jamais l'adresse de l'API en dur, et aucune configuration CORS n'est nécessaire.

## Secrets en local

Aucun secret n'est dans les dépôts :

- le mot de passe de la base est dans un fichier `.env` du backend, ignoré par Git, lu par Docker Compose ;
- la chaîne de connexion de l'API est dans les user-secrets .NET, stockés dans le dossier personnel, hors du dépôt.

Détail par environnement : [environnements](../deployment/environments.md#secrets-et-configuration).

## Vérifier l'installation

```bash
docker compose ps                                  # le service db est « healthy »
dotnet test                                        # tous les tests passent (Docker doit tourner)
curl http://localhost:5073/health                  # Healthy
curl http://localhost:5073/api/schedule            # programmation du jour, au format JSON
```

## Problèmes fréquents

| Symptôme | Cause probable | Solution |
|---|---|---|
| Les tests d'intégration échouent au démarrage | Docker n'est pas lancé | démarrer Docker, ou lancer les tests unitaires seuls : `dotnet test --filter "Category!=Integration"` |
| `dotnet ef` introuvable | outils locaux non restaurés | `dotnet tool restore` dans le backend |
| La programmation du jour est vide | séances de démonstration datées d'un autre jour | relancer `bash scripts/reset-demo-screenings.sh` |
| L'API ne trouve pas la base | user-secrets absents sur ce poste | refaire l'étape 3 du README du backend |
