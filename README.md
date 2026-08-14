# demanouromeo-quincallerie-app-backend
This is the backend of quincallerie app

## Deploiement sur Railway

Ce repo est pret pour un build Docker sur Railway (`Dockerfile` + `railway.json`,
builder `DOCKERFILE`). Pas de build local requis : Railway construit l'image a
partir du code pousse ici.

### 1. Base de donnees

Dans le projet Railway, ajouter un plugin **MySQL** (Provision MySQL). Railway
cree une base nommee `railway` par defaut et expose des variables internes
(`MYSQLHOST`, `MYSQLPORT`, `MYSQLUSER`, `MYSQLPASSWORD`, `MYSQLDATABASE`) sur
le service MySQL lui-meme.

### 2. Service backend

Creer un service a partir de ce repo GitHub (New → GitHub Repo). Dans l'onglet
**Variables** du service backend, definir :

| Variable | Valeur |
|---|---|
| `DB_HOST` | `${{MySQL.MYSQLHOST}}` |
| `DB_PORT` | `${{MySQL.MYSQLPORT}}` |
| `DB_USER` | `${{MySQL.MYSQLUSER}}` |
| `DB_PASSWORD` | `${{MySQL.MYSQLPASSWORD}}` |
| `DB_NAME` | `${{MySQL.MYSQLDATABASE}}` |
| `JWT_SECRET` | chaine aleatoire longue (ex. `openssl rand -base64 48`) |
| `JWT_EXPIRATION_MS` | `28800000` (8h) ou autre |
| `CORS_ALLOWED_ORIGINS` | `https://qmvogt.dmsacad.com` (separer par des virgules si plusieurs origines) |

Ne pas definir `PORT` : Railway l'injecte automatiquement au conteneur, et
`application.properties` le lit deja (`server.port=${PORT:${SERVER_PORT:8080}}`).

Flyway execute les migrations (`db/migration/V1`...`V6`) automatiquement au
demarrage du conteneur — aucune etape manuelle sur la base.

### 3. Premiere connexion

Au premier demarrage sur une base vide, `AdminSeeder` cree un compte
`admin` / `admin123` (role ADMIN). **Se connecter et changer ce mot de passe
immediatement** via le changement de mot de passe self-service
(`PUT /api/compte/mot-de-passe`) — ce compte par defaut n'est pas adapte a la
production tel quel.

### 4. Verification

Railway utilise `GET /actuator/health` comme healthcheck de deploiement
(configure dans `railway.json`) — expose sans authentification, il ne renvoie
que le statut (`{"status":"UP"}"`), aucun detail interne.
