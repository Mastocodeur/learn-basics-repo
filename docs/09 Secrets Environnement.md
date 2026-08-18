# Variables d'environnement et secrets

Les variables d'environnement permettent de séparer la configuration du code. Les secrets (clés API, mots de passe, tokens) ne doivent jamais être écrits en dur dans le code ni committés dans Git.

---

## Sommaire

- [Principe](#principe)
- [Le fichier .env](#le-fichier-env)
- [Le fichier .env.example](#le-fichier-envexample)
- [Lire les variables en Python](#lire-les-variables-en-python)
- [Ce qu'on ne commit jamais](#ce-quon-ne-commit-jamais)
- [Partager des secrets en équipe](#partager-des-secrets-en-équipe)
- [Bonnes pratiques](#bonnes-pratiques)

---

## Principe

La règle fondamentale : **le code est public, la configuration est privée**.

Un même code doit pouvoir tourner dans plusieurs environnements (local, staging, production) avec des configurations différentes. Les variables d'environnement portent ces différences — URLs de base de données, clés API, niveaux de log, flags de fonctionnalité — sans que le code change.

```
# À ne jamais faire
API_KEY = "sk-1234abcd..."

# À faire
import os
API_KEY = os.environ["API_KEY"]
```

---

## Le fichier .env

Le fichier `.env` est un fichier texte placé à la racine du projet qui contient les variables d'environnement pour votre environnement local. Il n'est **jamais commité**.

```bash
# .env
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
API_KEY=sk-1234abcd...
DEBUG=true
LOG_LEVEL=INFO
S3_BUCKET=mon-bucket-dev
```

### Syntaxe

```bash
# Commentaires avec #
NOM_VARIABLE=valeur

# Pas d'espaces autour du =
BONNE_SYNTAXE=valeur
MAUVAISE_SYNTAXE = valeur  # incorrect

# Les guillemets sont optionnels pour les valeurs simples
PORT=8000
NOM="Mon Application"

# Les valeurs multiligne nécessitent des guillemets
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
-----END RSA PRIVATE KEY-----"
```

---

## Le fichier .env.example

Le fichier `.env.example` est le jumeau public de `.env`. Il contient les mêmes variables mais **sans aucune valeur sensible**. Il est commité dans Git et sert de documentation pour les autres développeurs.

```bash
# .env.example
DATABASE_URL=postgresql://user:password@localhost:5432/mydb_dev
API_KEY=your_api_key_here
DEBUG=true
LOG_LEVEL=INFO
S3_BUCKET=your_bucket_name
```

Quand quelqu'un rejoint le projet :

```bash
# Copier le modèle
cp .env.example .env

# Remplir avec ses propres valeurs
# (récupérées auprès d'un collègue ou d'un gestionnaire de secrets)
```

---

## Lire les variables en Python

### Avec `os.environ` (bibliothèque standard)

```python
import os

# Lire une variable (lève KeyError si absente)
api_key = os.environ["API_KEY"]

# Lire avec une valeur par défaut
log_level = os.environ.get("LOG_LEVEL", "INFO")
debug = os.environ.get("DEBUG", "false").lower() == "true"
```

### Avec `python-dotenv` (recommandé en local)

`python-dotenv` charge automatiquement le fichier `.env` dans les variables d'environnement :

```bash
uv add python-dotenv
```

```python
from dotenv import load_dotenv
import os

# À appeler une seule fois, au démarrage de l'application
load_dotenv()

api_key = os.environ["API_KEY"]
database_url = os.environ["DATABASE_URL"]
```

> En production (Docker, serveur), les variables sont injectées directement par l'environnement d'exécution — `load_dotenv()` ne fait rien si les variables sont déjà définies, ce qui est le comportement attendu.

### Avec `pydantic-settings` (recommandé pour les projets plus structurés)

`pydantic-settings` permet de valider et typer les variables d'environnement :

```bash
uv add pydantic-settings
```

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    api_key: str
    database_url: str
    debug: bool = False
    log_level: str = "INFO"
    port: int = 8000

    class Config:
        env_file = ".env"

settings = Settings()
print(settings.api_key)   # typé, validé, chargé depuis .env
```

---

## Ce qu'on ne commit jamais

Les fichiers suivants doivent être dans le `.gitignore` et ne jamais apparaître dans un commit :

```gitignore
# Variables d'environnement
.env
.env.local
.env.*.local

# Clés et certificats
*.pem
*.key
*.p12
*.pfx
id_rsa
id_ed25519

# Fichiers de credentials
credentials.json
service_account.json
*_credentials.json
```

### Se protéger avec pre-commit

La meilleure protection contre un secret commité accidentellement est d'installer pre-commit avec le hook `detect-secrets` ou `detect-private-key`. Le commit est alors bloqué avant même d'atteindre Git — le problème est intercepté à la source.

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: detect-private-key     # bloque les clés privées SSH, PEM…
      - id: check-added-large-files # bloque les fichiers volumineux (datasets, dumps…)

  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets         # détecte les patterns de secrets (API keys, tokens…)
```

Avec cette configuration en place, commiter un fichier `.env` ou une clé privée est impossible — pre-commit refuse le commit et indique précisément ce qui a été détecté. C'est la raison principale pour laquelle pre-commit doit être installé sur tout projet dès le premier jour, indépendamment de la qualité du code.

Voir le [chapitre 06 — Pre-commit](06_Pre-commit.md) pour l'installation complète et le catalogue des hooks disponibles.

### Vérifier qu'un secret n'est pas dans l'historique Git

```bash
# Chercher une chaîne dans tout l'historique
git log --all --full-history -S "ma_cle_secrete" -- .
```

### Si un secret est trouvé dans l'historique

> **Première action, toujours** : révoquer le secret sur la plateforme concernée (GitHub, AWS, OpenAI…) avant même de toucher au code. Un secret dans l'historique Git est à considérer comme compromis, qu'il ait été vu ou non.

Ensuite, purger l'historique avec `git-filter-repo` (outil recommandé, plus sûr que `filter-branch`) :

```bash
# Installer git-filter-repo
pip install git-filter-repo

# Supprimer toutes les occurrences de la valeur sensible dans l'historique
git filter-repo --replace-text <(echo "ma_cle_secrete==>REMOVED")

# Forcer le push (l'historique a été réécrit)
git push origin --force --all
git push origin --force --tags
```

Après le push forcé, **toute personne ayant cloné le dépôt doit le re-cloner** — leurs copies locales contiennent encore l'ancien historique.

Si le dépôt est sur GitHub, ouvrir également `Settings → Danger Zone → Clear cached views` pour purger les caches de l'interface.

---

## Partager des secrets en équipe

Le fichier `.env` ne se partage pas par email ni par Slack. Les pratiques recommandées selon le niveau de maturité du projet :

**Pour démarrer** : un gestionnaire de mots de passe partagé (Bitwarden, 1Password Teams) avec un coffre dédié au projet.

**Pour les projets plus matures** : un gestionnaire de secrets dédié :
- [HashiCorp Vault](https://www.vaultproject.io/) — solution open source, auto-hébergeable
- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) — si l'infrastructure est sur AWS
- [Azure Key Vault](https://azure.microsoft.com/en-us/products/key-vault) — si l'infrastructure est sur Azure
- [Doppler](https://www.doppler.com/) — service SaaS simple à mettre en place

---

## Bonnes pratiques

**Nommer les variables de manière explicite**

```bash
# Bien : on sait à quoi ça sert
OPENAI_API_KEY=...
POSTGRES_DATABASE_URL=...
AWS_S3_BUCKET_NAME=...

# À éviter : trop vague
KEY=...
DB=...
BUCKET=...
```

**Préfixer par environnement si nécessaire**

```bash
# Pour distinguer les configurations par environnement
DEV_DATABASE_URL=...
PROD_DATABASE_URL=...
```

**Ne jamais logger les variables d'environnement**

```python
# À ne jamais faire
import os
print(os.environ)  # affiche tous les secrets dans les logs

# À faire si on doit déboguer
print(os.environ.get("LOG_LEVEL"))  # une variable précise, non sensible
```

**Valider les variables au démarrage**

L'application doit échouer immédiatement au démarrage si une variable requise est absente — et non au moment où elle est utilisée :

```python
import os

REQUIRED_VARS = ["API_KEY", "DATABASE_URL", "S3_BUCKET"]

missing = [var for var in REQUIRED_VARS if not os.environ.get(var)]
if missing:
    raise EnvironmentError(
        f"Variables d'environnement manquantes : {', '.join(missing)}\n"
        "Vérifiez votre fichier .env."
    )
```
