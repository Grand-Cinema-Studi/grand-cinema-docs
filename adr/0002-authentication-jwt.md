# ADR 0002 – Authentification : jeton JWT et mots de passe hachés par PBKDF2

- Statut : Accepté
- Concerne : les deux
- Liens : US 2.1 (issue #18), [sécurité](../security/security.md#authentification-et-rôles), Pull Request backend `feature/us-2-1-sign-up-and-sign-in`

## Contexte

- L'US 2.1 demande de créer un compte et de se connecter ; un mot de passe ne doit jamais être stocké en clair, et un échec de connexion ne doit pas dire si c'est l'adresse ou le mot de passe qui est faux.
- Quatre rôles existent (`Customer`, `Cashier`, `TicketInspector`, `Manager`) ; l'API doit vérifier le rôle à chaque appel, pas seulement le front.
- Le front Angular et l'API sont servis par le même proxy Caddy, sous la même adresse.
- L'API doit pouvoir passer à plusieurs instances pendant le Festival de Jazz, sans stockage de session partagé, avec un budget limité.
- Le mot de passe doit être haché par un algorithme lent et salé, sans bibliothèque payante ni dépendance lourde.

## Décision

1. **Session : jeton JWT signé (HMAC-SHA256)**, renvoyé par `POST /api/users` (inscription) et `POST /api/tokens` (connexion), puis envoyé par le front dans l'en-tête `Authorization: Bearer <jeton>`. Le jeton contient l'identifiant de l'utilisateur (`sub`), son adresse (`email`), son rôle (`role`) et expire au bout de 60 minutes. La clé de signature (32 octets au moins) est un secret : user-secrets en développement, Parameter Store en production.
2. **Hachage : `PasswordHasher` d'ASP.NET Core Identity** (PBKDF2, HMAC-SHA512, 100 000 itérations, sel aléatoire de 128 bits), utilisé seul, sans le reste d'ASP.NET Core Identity. Il est caché derrière l'interface `IPasswordHasher` de la couche Application.
3. **Accès par défaut refusé** : une politique de repli exige un utilisateur connecté sur tout endpoint qui ne déclare rien ; les endpoints publics le disent explicitement (`AllowAnonymous`).
4. **Limitation des tentatives** : 10 inscriptions ou connexions par minute et par adresse IP (limitation de débit d'ASP.NET Core), réponse `429` au-delà.
5. Mot de passe de 12 caractères au moins (recommandation de la CNIL pour une authentification par mot de passe seul), adresse enregistrée en minuscules.

## Alternatives écartées

- **Cookie de session `HttpOnly`** : il protège mieux le jeton contre le vol par XSS. Écarté pour le MVP : il demande une protection CSRF (jeton anti-falsification côté Angular et API), et un jeton dans un en-tête se teste directement dans Swagger et avec `curl`, ce qui sert la démonstration. À reconsidérer si le front stocke des données sensibles (voir « À surveiller »).
- **ASP.NET Core Identity complet** (tables, gestionnaires d'utilisateurs, pages) : sept tables supplémentaires et un modèle d'utilisateur imposé, pour des besoins couverts par une table `Users` et deux cas d'usage.
- **Fournisseur d'identité externe** (Auth0, Cognito, Keycloak) : payant au-delà d'un quota ou un serveur de plus à héberger ; surdimensionné pour le MVP.
- **Argon2** : plus résistant aux attaques par carte graphique, mais demande une bibliothèque tierce ; PBKDF2 reste recommandé par l'OWASP avec ce nombre d'itérations, et le format du hachage permet d'augmenter le coût plus tard sans invalider les mots de passe existants.

## Conséquences

- Positives : API sans état (aucune session en mémoire ni en base), donc plusieurs instances possibles sans changement ; rôle vérifié par l'API à chaque appel ; aucun mot de passe en clair, même dans les journaux ; tests unitaires possibles grâce aux interfaces `IPasswordHasher` et `IAccessTokenGenerator`.
- Négatives : un jeton émis reste valable jusqu'à son expiration (pas de révocation avant 60 minutes, pas de jeton de rafraîchissement dans le MVP) ; le front doit garder le jeton (en mémoire, ou en `sessionStorage`), donc rester protégé contre le XSS (échappement d'Angular, aucune injection de HTML).
- À surveiller : derrière le proxy Caddy, l'adresse IP vue par l'API est celle du proxy tant que les en-têtes `X-Forwarded-For` ne sont pas pris en compte (`UseForwardedHeaders`) : la limitation par IP devient alors globale. À régler à la mise en ligne de l'US 2.1.
