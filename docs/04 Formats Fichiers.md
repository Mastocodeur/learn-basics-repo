# Formats de fichiers

Dans un projet data, on manipule régulièrement des fichiers qui ne sont pas du code Python. Fichiers de configuration, fichiers de données, fichiers de documentation : chacun utilise un format spécifique avec sa propre syntaxe. Ce guide présente les formats les plus courants et explique comment les lire et les écrire.

---

## Sommaire

- [YAML](#yaml)
- [JSON](#json)
- [TOML](#toml)
- [Markdown](#markdown)
- [.env](#env)
- [CSV](#csv)
- [Parquet](#parquet)
- [Récapitulatif](#récapitulatif)

---

## YAML

**YAML** (à l'origine *Yet Another Markup Language*, puis renommé *YAML Ain't Markup Language* pour souligner que ce n'est pas un langage de balisage comme HTML ou XML) est un format de fichiers de configuration conçu pour être lisible par un humain. Il est très utilisé dans l'écosystème DevOps et en Python : fichiers de CI/CD (`.gitlab-ci.yml`), configuration de pre-commit (`.pre-commit-config.yaml`), fichiers Docker Compose, etc.

L'extension est `.yaml` ou `.yml` (les deux sont équivalentes).

### Syntaxe de base

YAML repose sur l'**indentation** (comme Python) pour représenter la hiérarchie des données. Il n'utilise ni accolades ni crochets pour structurer les données (contrairement à JSON).

Voici les éléments fondamentaux :

**Paires clé-valeur** — la brique de base de YAML. Une clé suivie de `:` et d'un espace, puis la valeur :

```yaml
name: mon-projet
version: 1.0
description: Un projet data science
```

**Types de données** — YAML reconnaît automatiquement les types courants sans avoir besoin de guillemets :

```yaml
# Chaîne de caractères
title: Mon projet

# Nombre entier
max_retries: 3

# Nombre décimal
learning_rate: 0.001

# Booléen (true/false, yes/no, on/off)
debug: true
verbose: false

# Valeur nulle
middle_name: null
```

> **Attention aux booléens :** YAML interprète `yes`, `no`, `on`, `off` comme des booléens, ce qui peut causer des surprises. Par exemple, la clé `country: no` sera interprétée comme `country: false` et non comme la chaîne `"no"` (pour la Norvège). En cas de doute, encadrer la valeur avec des guillemets : `country: "no"`.

**Listes** — représentées par des tirets `-` suivis d'un espace :

```yaml
# Liste simple
fruits:
  - pomme
  - banane
  - cerise

# Liste de nombres
scores:
  - 95
  - 87
  - 72
```

**Dictionnaires imbriqués** — on augmente l'indentation (2 espaces par convention) pour créer des niveaux hiérarchiques :

```yaml
database:
  host: localhost
  port: 5432
  credentials:
    username: admin
    password: secret
```

Ce qui correspond en Python à :

```python
{
    "database": {
        "host": "localhost",
        "port": 5432,
        "credentials": {
            "username": "admin",
            "password": "secret"
        }
    }
}
```

**Listes de dictionnaires** — très fréquent dans les fichiers de configuration. Chaque élément de la liste est un dictionnaire :

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.0
    hooks:
      - id: ruff
        args: [--fix]
```

Ici, `repos` est une liste contenant deux éléments. Chaque élément est un dictionnaire avec les clés `repo`, `rev` et `hooks`. La clé `hooks` contient elle-même une liste de dictionnaires.

**Chaînes multi-lignes** — YAML propose deux syntaxes pour les textes longs :

```yaml
# Avec `|` : conserve les retours à la ligne tels quels
description: |
  Ce projet est un pipeline de données
  qui traite les fichiers CSV
  et génère des rapports.

# Avec `>` : fusionne les lignes en un seul paragraphe
summary: >
  Ce projet est un pipeline de données
  qui traite les fichiers CSV
  et génère des rapports.
```

Avec `|`, le texte garde ses 3 lignes séparées. Avec `>`, le texte devient une seule ligne : *"Ce projet est un pipeline de données qui traite les fichiers CSV et génère des rapports."*

**Commentaires** — comme en Python, les commentaires commencent par `#` :

```yaml
# Configuration de la base de données
database:
  host: localhost  # Serveur local pour le développement
  port: 5432
```

### Pièges courants

| Piège | Problème | Solution |
|-------|----------|----------|
| Tabulations | YAML interdit les tabulations, uniquement des espaces | Configurer l'éditeur pour insérer des espaces |
| Booléens implicites | `yes`, `no`, `on`, `off` deviennent des booléens | Utiliser des guillemets : `"yes"`, `"no"` |
| Deux-points dans les valeurs | `title: Hello: World` provoque une erreur | Encadrer avec des guillemets : `title: "Hello: World"` |
| Indentation inconsistante | Mélanger 2 et 4 espaces casse la hiérarchie | Toujours utiliser 2 espaces par niveau |

### Où on le rencontre dans un projet

| Fichier | Rôle |
|---------|------|
| `.pre-commit-config.yaml` | Configuration des hooks de pre-commit |
| `.gitlab-ci.yml` | Pipeline de CI/CD GitLab |
| `docker-compose.yml` | Configuration des conteneurs Docker |
| `mkdocs.yml` | Configuration de la documentation MkDocs |

### Lire un fichier YAML en Python

```python
import yaml
from pathlib import Path

config = yaml.safe_load(Path("config.yaml").read_text())
print(config["database"]["host"])  # localhost
```

Le module `pyyaml` (installé via `uv add pyyaml`) fournit la fonction `yaml.safe_load()`. Toujours utiliser `safe_load()` et non `load()`, car ce dernier peut exécuter du code arbitraire contenu dans le fichier YAML.

---

## JSON

**JSON** (*JavaScript Object Notation*) est un format d'échange de données léger et universel. Bien qu'originaire de JavaScript, il est utilisé dans pratiquement tous les langages de programmation. C'est le format standard pour les réponses d'API web et les fichiers de configuration dans l'écosystème JavaScript.

L'extension est `.json`.

### Syntaxe de base

JSON est plus strict que YAML : les clés doivent être entre guillemets doubles, les virgules séparent les éléments, et il n'y a ni commentaires ni booléens implicites.

**Structure générale** — un fichier JSON est soit un objet (dictionnaire) entouré d'accolades `{}`, soit un tableau (liste) entouré de crochets `[]` :

```json
{
    "name": "mon-projet",
    "version": "1.0",
    "description": "Un projet data science",
    "debug": true,
    "max_retries": 3,
    "learning_rate": 0.001,
    "middle_name": null
}
```

**Types de données** — JSON supporte 6 types :

| Type | Exemple | Équivalent Python |
|------|---------|-------------------|
| Chaîne de caractères | `"hello"` | `str` |
| Nombre entier | `42` | `int` |
| Nombre décimal | `3.14` | `float` |
| Booléen | `true`, `false` | `bool` |
| Null | `null` | `None` |
| Tableau | `[1, 2, 3]` | `list` |
| Objet | `{"key": "value"}` | `dict` |

**Imbrication** — les objets et tableaux peuvent être imbriqués sans limite :

```json
{
    "database": {
        "host": "localhost",
        "port": 5432,
        "credentials": {
            "username": "admin",
            "password": "secret"
        }
    },
    "features": ["login", "dashboard", "export"],
    "users": [
        {"name": "Alice", "role": "admin"},
        {"name": "Bob", "role": "developer"}
    ]
}
```

### Différences avec YAML

| Critère | JSON | YAML |
|---------|------|------|
| Lisibilité | Moins lisible (accolades, guillemets obligatoires) | Plus lisible (indentation, pas de guillemets) |
| Commentaires | Non supportés | Supportés (`#`) |
| Taille | Plus verbeux | Plus compact |
| Parsing | Plus rapide et plus strict | Plus lent, plus permissif |
| Usage principal | Échange de données (API, web) | Fichiers de configuration |

En résumé : JSON est fait pour être lu par des machines, YAML est fait pour être lu par des humains.

### Où on le rencontre dans un projet

| Fichier | Rôle |
|---------|------|
| `package.json` | Métadonnées d'un projet JavaScript/Node.js |
| `tsconfig.json` | Configuration TypeScript |
| Réponses d'API | Format standard des API REST |
| Fichiers de données | Export/import de données structurées |

### Lire et écrire du JSON en Python

Python intègre nativement le module `json` (pas besoin d'installer de bibliothèque) :

```python
import json
from pathlib import Path

# Lire
data = json.loads(Path("data.json").read_text())

# Écrire (indent=2 pour la lisibilité, ensure_ascii=False pour les accents)
Path("output.json").write_text(
    json.dumps(data, indent=2, ensure_ascii=False)
)
```

---

## TOML

**TOML** (*Tom's Obvious Minimal Language*) est un format de configuration conçu pour être simple et non ambigu. Il est devenu le standard en Python depuis l'adoption du `pyproject.toml` ([PEP 621](https://peps.python.org/pep-0621/)).

L'extension est `.toml`.

### Syntaxe de base

TOML ressemble à un fichier `.ini` amélioré. Il utilise des sections entre crochets `[]` pour organiser les données, et des paires `clé = valeur` à l'intérieur de chaque section.

**Paires clé-valeur** :

```toml
name = "mon-projet"
version = "1.0"
debug = true
max_retries = 3
learning_rate = 0.001
```

**Sections** — les crochets `[]` définissent des groupes. Chaque section correspond à un dictionnaire imbriqué :

```toml
[project]
name = "mon-projet"
version = "1.0"

[project.authors]
name = "Alice"
email = "alice@example.com"

[tool.ruff]
line-length = 120
select = ["E", "F", "I", "W"]
```

Ce qui correspond en Python à :

```python
{
    "project": {
        "name": "mon-projet",
        "version": "1.0",
        "authors": {
            "name": "Alice",
            "email": "alice@example.com"
        }
    },
    "tool": {
        "ruff": {
            "line-length": 120,
            "select": ["E", "F", "I", "W"]
        }
    }
}
```

**Tableaux** — les listes sont entre crochets, comme en JSON :

```toml
dependencies = [
    "pandas>=2.0",
    "scikit-learn>=1.3",
    "numpy>=1.24",
]
```

**Tableaux de tables** — les doubles crochets `[[]]` créent des listes de dictionnaires :

```toml
[[project.authors]]
name = "Alice"
email = "alice@example.com"

[[project.authors]]
name = "Bob"
email = "bob@example.com"
```

Ce qui donne en Python : `{"project": {"authors": [{"name": "Alice", ...}, {"name": "Bob", ...}]}}`

**Commentaires** — avec `#`, comme en Python et YAML :

```toml
# Configuration du linter
[tool.ruff]
line-length = 120  # Longueur maximale d'une ligne
```

### Différences avec YAML et JSON

| Critère | TOML | YAML | JSON |
|---------|------|------|------|
| Lisibilité | Très lisible (style `.ini`) | Lisible (indentation) | Moins lisible (accolades) |
| Commentaires | Oui (`#`) | Oui (`#`) | Non |
| Types | Strict (dates, heures natifs) | Permissif (booléens implicites) | Strict |
| Ambiguïté | Aucune | Possible (booléens, types) | Aucune |
| Usage principal | Configuration Python | Configuration DevOps | Échange de données |

Le principal avantage de TOML sur YAML : il est **non ambigu**. En YAML, `port: 8080` pourrait être un nombre ou une chaîne selon le contexte. En TOML, `port = 8080` est toujours un nombre entier, et `port = "8080"` est toujours une chaîne.

### Où on le rencontre dans un projet

| Fichier | Rôle |
|---------|------|
| `pyproject.toml` | Métadonnées, dépendances et configuration des outils Python |
| `ruff.toml` | Configuration de Ruff (alternative à `[tool.ruff]` dans `pyproject.toml`) |
| `Cargo.toml` | Métadonnées d'un projet Rust |

### Lire un fichier TOML en Python

Depuis Python 3.11, le module `tomllib` est intégré nativement :

```python
import tomllib
from pathlib import Path

config = tomllib.loads(Path("pyproject.toml").read_text())
print(config["project"]["name"])  # mon-projet
```

Pour les versions antérieures, installer le package `tomli` via `uv add tomli`.

---

## Markdown

**Markdown** est un format de balisage léger permettant de rédiger du texte formaté (titres, listes, liens, code) avec une syntaxe simple et lisible même sans rendu. C'est le format standard pour la documentation dans les projets logiciels.

L'extension est `.md`.

### Syntaxe de base

**Titres** — de 1 à 6 niveaux, définis par le nombre de `#` :

```markdown
# Titre de niveau 1
## Titre de niveau 2
### Titre de niveau 3
```

**Texte formaté** :

```markdown
**gras**
*italique*
`code en ligne`
~~barré~~
```

**Listes** :

```markdown
- Élément non ordonné
- Autre élément
  - Sous-élément (2 espaces d'indentation)

1. Premier élément ordonné
2. Deuxième élément
```

**Liens et images** :

```markdown
[Texte du lien](https://example.com)
![Texte alternatif](chemin/vers/image.png)
```

**Blocs de code** — entourés de triple backticks, avec le langage optionnel pour la coloration syntaxique :

````markdown
```python
def hello():
    print("Bonjour")
```
````

**Tableaux** :

```markdown
| Colonne 1 | Colonne 2 |
|-----------|-----------|
| Cellule   | Cellule   |
```

**Citations** :

```markdown
> Ceci est une citation.
```

### Où on le rencontre dans un projet

| Fichier | Rôle |
|---------|------|
| `README.md` | Présentation du projet (premier fichier lu) |
| `CONTRIBUTING.md` | Règles de contribution |
| `CHANGELOG.md` | Historique des modifications par version |
| `docs/*.md` | Pages de documentation |
| Fichiers wiki (GitLab/GitHub) | Pages du wiki du projet |

---

## .env

Le format **`.env`** (*environment*) est un format minimaliste pour stocker des variables d'environnement. Chaque ligne définit une paire `CLÉ=valeur`. Ce format est utilisé pour séparer la configuration sensible (mots de passe, clés API) du code source.

L'extension est `.env` (le fichier entier s'appelle `.env`, sans nom avant le point).

### Syntaxe de base

```env
# Commentaire
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
API_KEY=sk-1234567890abcdef
DEBUG=true
SECRET_KEY=ma_cle_secrete

# Les guillemets sont optionnels, utiles pour les valeurs avec espaces
APP_NAME="Mon Application"
```

**Règles :**
- Pas d'espaces autour du `=`
- Les commentaires commencent par `#`
- Les guillemets sont optionnels (utiles si la valeur contient des espaces)
- Les clés sont en `UPPER_SNAKE_CASE` par convention

### Lire un fichier .env en Python

Le package `python-dotenv` (installé via `uv add python-dotenv`) charge automatiquement les variables dans l'environnement :

```python
from dotenv import load_dotenv
import os

load_dotenv()  # Charge les variables du fichier .env

db_url = os.getenv("DATABASE_URL")
api_key = os.getenv("API_KEY")
```

### Point de sécurité

Le fichier `.env` contient des secrets et ne doit **jamais** être versionné dans Git. C'est pourquoi il est systématiquement présent dans le `.gitignore`. Le fichier `.env.example`, qui contient les mêmes clés mais avec des valeurs fictives, sert de template et lui est versionné (cf. [Structure d'un projet](03 Structure Projet.md)).

---

## CSV

**CSV** (*Comma-Separated Values*) est le format tabulaire le plus répandu. Chaque ligne représente un enregistrement, et les valeurs sont séparées par un caractère délimiteur (virgule, point-virgule ou tabulation).

L'extension est `.csv` ou `.tsv` (pour les fichiers séparés par des tabulations).

### Syntaxe de base

```csv
nom,age,ville
Alice,30,Paris
Bob,25,Lyon
Charlie,35,Marseille
```

Avec un séparateur point-virgule (fréquent dans les fichiers français, car la virgule est utilisée comme séparateur décimal) :

```csv
nom;age;ville
Alice;30;Paris
Bob;25;Lyon
```

### Lire un CSV en Python

```python
import pandas as pd

# Séparateur virgule (par défaut)
df = pd.read_csv("data.csv")

# Séparateur point-virgule
df = pd.read_csv("data.csv", sep=";")

# Avec encodage spécifique (fréquent pour les fichiers français)
df = pd.read_csv("data.csv", sep=";", encoding="latin-1")
```

### Limites du CSV

Le format CSV est simple mais présente plusieurs inconvénients :

- **Pas de typage** : toutes les valeurs sont des chaînes de caractères, il faut les convertir manuellement
- **Pas de standard strict** : le délimiteur, l'encodage et le format des dates varient selon les outils qui ont généré le fichier
- **Performance** : la lecture est lente sur les fichiers volumineux car le fichier est entièrement textuel
- **Taille** : les fichiers sont plus volumineux que les formats binaires comme Parquet

Pour les fichiers volumineux ou les pipelines de données, le format Parquet est préférable.

---

## Parquet

**Parquet** est un format de fichier **binaire** et **colonnaire**, conçu pour le stockage et le traitement efficace de gros volumes de données. Contrairement au CSV qui stocke les données ligne par ligne en texte, Parquet compresse les données par colonne, ce qui le rend beaucoup plus rapide et compact.

L'extension est `.parquet`.

### Pourquoi Parquet plutôt que CSV ?

| Critère | CSV | Parquet |
|---------|-----|---------|
| Format | Texte | Binaire |
| Taille sur disque | Volumineux | 2x à 10x plus petit (compression) |
| Vitesse de lecture | Lente | Rapide (lecture sélective par colonne) |
| Typage des données | Non (tout est texte) | Oui (int, float, string, date, etc.) |
| Lisible par un humain | Oui (ouvrable dans un éditeur texte) | Non (nécessite un outil) |
| Standard | Pas de standard strict | Spécification unique et rigoureuse |

En pratique, Parquet est le format de référence pour les pipelines de données. On utilise CSV pour les échanges ponctuels ou les petits fichiers, et Parquet dès que les données dépassent quelques milliers de lignes ou sont traitées de manière récurrente.

### Lire et écrire du Parquet en Python

```python
import pandas as pd

# Lire
df = pd.read_parquet("data.parquet")

# Écrire
df.to_parquet("output.parquet", index=False)

# Lire uniquement certaines colonnes (avantage du format colonnaire)
df = pd.read_parquet("data.parquet", columns=["nom", "age"])
```

Le dernier exemple illustre l'avantage principal du format colonnaire : on peut lire uniquement les colonnes nécessaires sans charger tout le fichier en mémoire. Avec un CSV, le fichier entier est lu même si on n'a besoin que de 2 colonnes sur 50.

---

## Récapitulatif

| Format | Extension | Nature | Usage principal | Lisible par un humain |
|--------|-----------|--------|-----------------|----------------------|
| **YAML** | `.yaml`, `.yml` | Texte | Configuration (CI/CD, pre-commit, Docker) | Oui |
| **JSON** | `.json` | Texte | Échange de données (API, web) | Oui |
| **TOML** | `.toml` | Texte | Configuration Python (`pyproject.toml`) | Oui |
| **Markdown** | `.md` | Texte | Documentation (README, wiki) | Oui |
| **.env** | `.env` | Texte | Variables d'environnement (secrets) | Oui |
| **CSV** | `.csv` | Texte | Données tabulaires (petits volumes, échanges) | Oui |
| **Parquet** | `.parquet` | Binaire | Données tabulaires (gros volumes, pipelines) | Non |
