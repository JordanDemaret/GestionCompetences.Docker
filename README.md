# GestionCompetences - Docker

Ce dépôt contient la configuration Docker permettant de lancer l'ensemble de l'application **GestionCompetences** :

* 🖥️ Frontend Angular
* ⚙️ Backend ASP.NET Core
* 🗄️ SQL Server 2022

Le frontend et le backend sont stockés dans des dépôts Git séparés.

---

## 📁 Structure des dépôts

Les trois projets doivent être placés dans le même dossier :

```text
Projet/
│
├── GestionCompetences/
│   ├── Dockerfile
│   └── ...
│
├── GestionCompetences.Angular/
│   ├── Dockerfile
│   └── ...
│
└── GestionCompetences-Docker/
    ├── docker-compose.yml
    ├── .env
    ├── .env.example
    ├── .gitignore
    └── README.md
```

Le `docker-compose.yml` utilise les Dockerfiles des deux autres projets.

---

## 🏗️ Architecture

```text
                    Docker
┌──────────────────────────────────────────────┐
│                                              │
│   ┌──────────────────────┐                   │
│   │       Frontend       │                   │
│   │       Angular        │                   │
│   │      Port : 4200     │                   │
│   └──────────┬───────────┘                   │
│              │                                │
│              ▼                                │
│   ┌──────────────────────┐                   │
│   │       Backend        │                   │
│   │     ASP.NET Core     │                   │
│   │      Port : 7281     │                   │
│   └──────────┬───────────┘                   │
│              │                                │
│              ▼                                │
│   ┌──────────────────────┐                   │
│   │      SQL Server      │                   │
│   │       Port : 1433    │                   │
│   └──────────────────────┘                   │
│                                              │
│              app-net                         │
└──────────────────────────────────────────────┘
```

---

# 🔧 Prérequis

Installer :

* Docker Desktop
* Git

Vérifier l'installation :

```bash
docker --version
docker compose version
```

---

# 📥 Installation

## 1. Cloner les dépôts

Les trois dépôts doivent être placés dans le même dossier.

Exemple :

```bash
git clone https://github.com/JordanDemaret/GestionCompetences.git GestionCompetences
git clone https://github.com/JordanDemaret/GestionCompetences.Angular.git GestionCompetences.Angular
git clone https://github.com/JordanDemaret/GestionCompetences.Docker.git GestionCompetences-Docker
```

Structure finale :

```text
Projet/
├── GestionCompetences/
├── GestionCompetences.Angular/
└── GestionCompetences-Docker/
```

---

# 🔐 Configuration des variables d'environnement

Les informations sensibles ne sont pas stockées directement dans le `docker-compose.yml`.

Créer un fichier `.env` dans le dossier du projet Docker :

```env
SA_PASSWD=VotreMotDePasseSQL
JWT_KEY=VotreCleSecreteJWT
```

# 🐳 Lancer l'application

Se placer dans le dossier Docker :

```bash
cd GestionCompetences-Docker
```

Puis lancer :

```bash
docker compose up --build
```

Pour lancer les conteneurs en arrière-plan :

```bash
docker compose up -d --build
```

Docker va démarrer :

1. SQL Server
2. Backend ASP.NET Core
3. Frontend Angular

---

# 📊 Vérifier les conteneurs

Utiliser :

```bash
docker compose ps
```

Les trois services doivent être présents :

```text
mssql-server
Competence-Api
angular_frontend
```

Pour voir les logs :

```bash
docker compose logs
```

---

# 🌐 Accès aux services

## Frontend Angular

Le frontend est disponible à :

```text
http://localhost:4200
```

## Backend ASP.NET Core

L'API est disponible à :

```text
http://localhost:7281/scalar/
```

## SQL Server

SQL Server est exposé sur :

```text
localhost:1434
```

Le port interne utilisé entre le backend et SQL Server reste :

```text
1433
```

La connexion du backend utilise donc :

```text
Server=mssql,1433
```

`mssql` correspond au nom du service Docker Compose.

---

# 🔗 Communication entre les conteneurs

Les services utilisent le réseau Docker :

```text
app-net
```

Le backend communique avec SQL Server grâce au nom du service :

```text
mssql
```

La chaîne de connexion est donc de la forme :

```text
Server=mssql,1433;Database=GestionCompetencesDb;User Id=sa;Password=...;TrustServerCertificate=True;
```

Il ne faut pas utiliser :

```text
Server=localhost,1433
```

depuis le conteneur Backend.

---

# 🛑 Arrêter l'application

Pour arrêter les conteneurs :

```bash
docker compose down
```

Cette commande supprime les conteneurs mais conserve les données SQL stockées dans le volume Docker.

---

# ⚠️ Supprimer les données SQL

Pour supprimer également le volume SQL :

```bash
docker compose down -v
```

⚠️ Cette commande supprime les données de la base de données stockées dans le volume `sqlserver_data`.

À utiliser uniquement si vous souhaitez repartir avec une base de données vierge.

---

# 🔄 Reconstruire les conteneurs

Après une modification du code ou d'un Dockerfile :

```bash
docker compose up -d --build
```

Pour forcer la recréation des conteneurs :

```bash
docker compose up -d --build --force-recreate
```

---

# 📋 Logs

## Tous les services

```bash
docker compose logs
```

## SQL Server

```bash
docker logs mssql-server
```

## Backend

```bash
docker logs Competence-Api
```

## Frontend

```bash
docker logs angular_frontend
```

Pour suivre les logs en temps réel :

```bash
docker compose logs -f
```

---

# 💾 Volume SQL Server

SQL Server utilise le volume Docker :

```text
sqlserver_data
```

Ce volume permet de conserver les données SQL même lorsque les conteneurs sont supprimés.

Lister les volumes :

```bash
docker volume ls
```

---

# 🔒 Sécurité

Les informations sensibles doivent rester dans `.env` :

```env
SA_PASSWD=...
JWT_KEY=...
```

Le fichier `.env` **ne doit jamais être commité dans Git**.

Le fichier `.env.example` peut être commité avec des valeurs fictives :

```env
SA_PASSWD=CHANGE_ME
JWT_KEY=CHANGE_ME
```

Ne jamais publier :

* mot de passe SQL Server
* clé JWT
* autres clés ou secrets
* fichier `.env`

---

# 🧹 Nettoyage

Pour arrêter et supprimer les conteneurs :

```bash
docker compose down
```

Pour supprimer également le volume SQL :

```bash
docker compose down -v
```

Pour supprimer les images inutilisées :

```bash
docker image prune
```

---

# 🛠️ Technologies

* Docker
* Docker Compose
* ASP.NET Core
* Angular
* Microsoft SQL Server 2022

---

# 👨‍💻 Projet

**GestionCompetences**

Le projet est composé de trois parties :

* **GestionCompetences** : Backend ASP.NET Core
* **GestionCompetences.Angular** : Frontend Angular
* **GestionCompetences-Docker** : Configuration Docker Compose
