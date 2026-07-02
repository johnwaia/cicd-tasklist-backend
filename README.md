# CICD Tasklist — Backend

API REST pour la gestion de tâches (tasklist), construite avec Express, TypeScript et Prisma (MySQL).

## Stack technique

- **Runtime** : Node.js 20
- **Framework** : Express 5
- **ORM** : Prisma 6 (MySQL)
- **Langage** : TypeScript
- **Tests** : Vitest + Supertest (tests unitaires et e2e)
- **Qualité de code** : SonarQube
- **CI/CD** : Jenkins
- **Conteneurisation** : Docker

## Prérequis

- Node.js >= 20
- npm
- Une base MySQL accessible (locale ou via Docker)
- Docker (optionnel, pour le lancement conteneurisé)

## Installation

```bash
npm install
```

## Configuration

Crée un fichier `.env` à la racine du projet avec les variables suivantes :

```env
DATABASE_URL=mysql://root@localhost:3306/tasklist
PORT=3001
```

| Variable       | Description                                  | Défaut |
|----------------|-----------------------------------------------|--------|
| `DATABASE_URL` | URL de connexion MySQL (format Prisma)        | —      |
| `PORT`         | Port d'écoute du serveur HTTP                 | `3001` |

## Base de données

Le schéma est défini dans [prisma/schema.prisma](prisma/schema.prisma) avec un modèle unique `Task` :

| Champ         | Type       | Détails                          |
|---------------|------------|-----------------------------------|
| `id`          | Int        | Clé primaire, auto-incrémentée   |
| `title`       | String     | Requis                           |
| `description` | String?    | Optionnel                        |
| `completed`   | Boolean    | Défaut : `false`                 |
| `createdAt`   | DateTime   | Auto-généré                      |
| `updatedAt`   | DateTime   | Auto-mis à jour                  |

Générer le client Prisma :

```bash
npm run prisma:generate
```

Appliquer les migrations (environnement de dev) :

```bash
npm run prisma:migrate
```

## Lancer le projet

### En développement (avec rechargement à chaud)

```bash
npm run dev
```

### En production

```bash
npm run build
npm start
```

Le serveur démarre sur `http://localhost:3001` (ou le `PORT` configuré).

### Avec Docker Compose (recommandé)

Depuis la racine du dépôt principal (qui contient `docker-compose.yml`) :

```bash
docker compose up -d
```

Cela démarre la base MySQL, le backend, le frontend, et SonarQube.

## Tests

```bash
npm test                  # Tests unitaires
npm run test:coverage     # Tests unitaires + couverture
npm run test:e2e          # Tests end-to-end
npm run test:e2e:coverage # Tests e2e + couverture
```

Les rapports de couverture sont générés au format `lcov` (utilisé par SonarQube) et les résultats de tests au format JUnit (`reports/junit.xml`).

## API

Toutes les routes sont préfixées par `/api/tasks`.

| Méthode | Route             | Description                  |
|---------|-------------------|-------------------------------|
| GET     | `/api/tasks`      | Liste toutes les tâches      |
| GET     | `/api/tasks/:id`  | Récupère une tâche par ID    |
| POST    | `/api/tasks`      | Crée une nouvelle tâche      |
| PUT     | `/api/tasks/:id`  | Met à jour une tâche         |
| DELETE  | `/api/tasks/:id`  | Supprime une tâche           |

### Exemple — Créer une tâche

```bash
curl -X POST http://localhost:3001/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Faire les courses", "description": "Lait, oeufs, pain"}'
```

### Codes de réponse

- `200` / `201` : succès
- `204` : suppression réussie (pas de contenu)
- `400` : requête invalide (ex: ID non numérique, titre manquant)
- `404` : tâche non trouvée
- `500` : erreur serveur

## Docker

Le [Dockerfile](Dockerfile) utilise un build multi-stage :

1. **builder** : installe les dépendances, génère le client Prisma, compile le TypeScript
2. **runner** : image allégée (production uniquement) qui exécute `dist/server.js`

Build manuel de l'image :

```bash
docker build -t cicd-tasklist-backend .
docker run -p 3001:3001 --env-file .env cicd-tasklist-backend
```

## CI/CD (Jenkins)

Le [Jenkinsfile](Jenkinsfile) définit le pipeline suivant :

1. **SCM** : checkout du dépôt
2. **Install & Test** : `npm ci` + tests avec couverture
3. **SonarQube Analysis** : analyse de qualité de code via SonarScanner
4. **Docker Build** : build de l'image taguée `latest` et `<BUILD_NUMBER>`
5. **Docker Push** : publication sur DockerHub (`johnwaia/cicd-tasklist-backend`)

### Prérequis Jenkins

- Outil **Node20** configuré (NodeJS plugin)
- Outil **SonarScanner** configuré
- Serveur SonarQube déclaré dans **Manage Jenkins → System → SonarQube servers**
- Credential **Secret text** pour le token SonarQube
- Credential **Username with password** `dockerhub-credentials` pour DockerHub
- Docker accessible depuis l'agent Jenkins

## Structure du projet

```
cicd-tasklist-backend/
├── prisma/
│   └── schema.prisma          # Schéma de base de données
├── src/
│   ├── controllers/           # Logique des routes HTTP
│   ├── services/              # Logique métier / accès aux données
│   ├── routes/                # Définition des routes Express
│   ├── lib/                   # Utilitaires (client Prisma, etc.)
│   ├── __tests__/             # Tests unitaires et e2e
│   ├── app.ts                 # Configuration Express
│   └── server.ts              # Point d'entrée
├── Dockerfile
├── Jenkinsfile
└── package.json
```
