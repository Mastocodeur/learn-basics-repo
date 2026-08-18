# Docker

Docker permet d'emballer une application avec tout son environnement (Python, dépendances, variables de configuration) dans une image portable qui s'exécute de manière identique partout : sur votre machine, celle d'un collègue, ou un serveur de production.

---

## Sommaire

- [Concepts clés](#concepts-clés)
- [Installation](#installation)
- [Dockerfile](#dockerfile)
- [Commandes essentielles](#commandes-essentielles)
- [Docker Compose](#docker-compose)
- [Bonnes pratiques](#bonnes-pratiques)
- [Cas d'usage data / ML](#cas-dusage-data--ml)

---

## Concepts clés

| Terme | Définition |
|---|---|
| **Image** | Le modèle immuable — comme une recette. Elle contient l'OS, Python, les dépendances et le code. |
| **Conteneur** | Une instance en cours d'exécution d'une image. On peut en lancer plusieurs depuis la même image. |
| **Dockerfile** | Le fichier texte qui décrit comment construire l'image, étape par étape. |
| **Registry** | Un dépôt d'images (Docker Hub, GitHub Container Registry, AWS ECR…). |
| **Docker Compose** | Un outil pour orchestrer plusieurs conteneurs qui fonctionnent ensemble. |

---

## Installation

Docker Desktop : https://www.docker.com/products/docker-desktop/

Vérification après installation :

```bash
docker --version
docker compose version
```

---

## Dockerfile

Un `Dockerfile` décrit la construction d'une image. Voici un exemple typique pour un projet Python avec `uv` :

```dockerfile
# Image de base officielle Python (version slim = plus légère)
FROM python:3.12-slim

# Installer uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

# Répertoire de travail dans le conteneur
WORKDIR /app

# Copier les fichiers de dépendances en premier
# (optimise le cache Docker : les dépendances ne se réinstallent
# que si pyproject.toml ou uv.lock changent)
COPY pyproject.toml uv.lock ./

# Installer uniquement les dépendances de production
RUN uv sync --frozen --no-dev

# Copier le reste du code source
COPY src/ ./src/

# Commande exécutée au démarrage du conteneur
CMD ["uv", "run", "python", "src/mon_projet/main.py"]
```

### Instructions principales

| Instruction | Rôle |
|---|---|
| `FROM` | Image de base sur laquelle on construit |
| `WORKDIR` | Répertoire de travail dans le conteneur |
| `COPY` | Copie des fichiers depuis la machine hôte vers l'image |
| `RUN` | Exécute une commande pendant la construction de l'image |
| `ENV` | Définit une variable d'environnement |
| `EXPOSE` | Documente le port écouté (ne l'ouvre pas) |
| `CMD` | Commande par défaut au démarrage du conteneur |

---

## Commandes essentielles

```bash
# Construire une image depuis le Dockerfile du répertoire courant
docker build -t mon-projet:latest .

# Lancer un conteneur
docker run mon-projet:latest

# Lancer en mode interactif (avec un terminal)
docker run -it mon-projet:latest bash

# Lancer en arrière-plan
docker run -d mon-projet:latest

# Mapper un port (hôte:conteneur)
docker run -p 8080:8000 mon-projet:latest

# Monter un dossier local dans le conteneur
docker run -v $(pwd)/data:/app/data mon-projet:latest

# Passer des variables d'environnement
docker run --env-file .env mon-projet:latest

# Lister les conteneurs en cours d'exécution
docker ps

# Arrêter un conteneur
docker stop <container_id>

# Voir les logs d'un conteneur
docker logs <container_id>

# Supprimer les images et conteneurs inutilisés
docker system prune
```

---

## Docker Compose

Docker Compose permet de définir et lancer plusieurs services liés (une API, une base de données, un cache…) dans un seul fichier `docker-compose.yml`.

```yaml
services:

  app:
    build: .
    ports:
      - "8000:8000"
    env_file:
      - .env
    volumes:
      - ./data:/app/data
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

```bash
# Lancer tous les services
docker compose up

# Lancer en arrière-plan
docker compose up -d

# Arrêter tous les services
docker compose down

# Reconstruire les images avant de lancer
docker compose up --build

# Voir les logs de tous les services
docker compose logs -f
```

---

## Bonnes pratiques

**Utiliser des images slim ou alpine**

```dockerfile
# Bien : image légère
FROM python:3.12-slim

# À éviter : image complète (inutilement lourde)
FROM python:3.12
```

**Ordonner les instructions du plus stable au plus volatile**

Docker met en cache chaque couche. Si une instruction change, toutes celles qui suivent sont reconstruites. Copier les fichiers de dépendances avant le code source permet de ne pas réinstaller les packages à chaque modification du code.

```dockerfile
# Bien : dépendances d'abord, code ensuite
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev
COPY src/ ./src/

# À éviter : tout copier d'un coup
COPY . .
RUN uv sync --frozen --no-dev
```

**Ne jamais stocker de secrets dans l'image**

```dockerfile
# À ne jamais faire
ENV API_KEY=ma_cle_secrete

# À faire : passer les secrets au runtime via --env-file
docker run --env-file .env mon-projet:latest
```

**Ajouter un `.dockerignore`**

Comme `.gitignore`, ce fichier empêche de copier des fichiers inutiles dans l'image :

```
.venv/
.git/
.env
__pycache__/
*.pyc
data/
notebooks/
*.egg-info/
```

**Utiliser des builds multi-étapes pour les images de production**

```dockerfile
# Étape 1 : construction
FROM python:3.12-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev

# Étape 2 : image finale (sans les outils de build)
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /app/.venv ./.venv
COPY src/ ./src/
CMD [".venv/bin/python", "src/mon_projet/main.py"]
```

---

## Cas d'usage data / ML

**Exposer un notebook Jupyter**

```bash
docker run -p 8888:8888 -v $(pwd):/app jupyter/scipy-notebook
```

**Générer un `requirements.txt` pour Docker**

Si votre Dockerfile n'utilise pas `uv`, vous pouvez générer un `requirements.txt` depuis votre environnement local :

```bash
uv pip freeze > requirements.txt
```

Puis dans le Dockerfile :

```dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

**Séparer les données du code**

Les données ne doivent jamais être dans l'image. Utiliser des volumes ou des montages :

```bash
# Monter un dossier de données local
docker run -v $(pwd)/data:/app/data mon-projet:latest
```

Ou dans `docker-compose.yml`, utiliser un volume nommé monté sur un stockage externe (S3, NFS…).
