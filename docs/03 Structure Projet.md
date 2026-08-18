# Structure d'un projet

Ce guide décrit la structure type d'un projet Python data science tel que généré par le [Cookiecutter Data Science](https://github.com/drivendataorg/cookiecutter-data-science). Il détaille le rôle de chaque fichier de configuration et de chaque dossier.

Pour la création d'un projet à partir de ce template, se référer au guide : [Configuration d'un projet](<05 Routine Projet.md>).

---

## Sommaire

- [Vue d'ensemble](#vue-densemble)
- [Fichiers de configuration](#fichiers-de-configuration)
  - [`pyproject.toml`](#pyprojecttoml)
  - [`uv.lock`](#uvlock)
  - [`.python-version`](#python-version)
  - [`.gitignore`](#gitignore)
  - [`.pre-commit-config.yaml`](#pre-commit-configyaml)
  - [`.env.example`](#envexample)
  - [`Makefile`](#makefile)
  - [`LICENSE` et `CONTRIBUTING.md`](#license-et-contributingmd--deux-fichiers-à-ne-pas-confondre)
- [Dossiers du projet](#dossiers-du-projet)
  - [`src/`](#src)
  - [`data/`](#data)
  - [`tests/`](#tests)
  - [`notebooks/`](#notebooks)
  - [`docs/`](#docs)
  - [`.gitlab/` / `.azuredevops/`](#gitlab--azuredevops)
  - [`Dockerfile` et `.dockerignore`](#dockerfile-et-dockerignore)

---

## Vue d'ensemble

```
mon-projet/
├── .git/                        # Historique Git
├── .gitignore                   # Fichiers exclus du versionnement
├── .dockerignore                # Fichiers exclus de l'image Docker
├── .pre-commit-config.yaml      # Hooks de vérification avant chaque commit
├── .python-version              # Version de Python utilisée
├── .env.example                 # Template des variables d'environnement
├── pyproject.toml               # Métadonnées du projet et dépendances
├── uv.lock                      # Verrouillage exact des dépendances
├── Dockerfile                   # Instructions de construction de l'image Docker
├── Makefile                     # Commandes raccourcies (make test, make lint…)
├── LICENSE                      # Licence du projet (droits légaux)
├── CONTRIBUTING.md              # Règles de contribution au projet
├── README.md                    # Présentation du projet
│
├── src/                         # Code source principal
│   └── mon_projet/
│       ├── __init__.py
│       ├── main.py
│       └── utils.py
│
├── data/                        # Données du projet
│   ├── raw/                     # Données brutes (non modifiées)
│   ├── processed/               # Données nettoyées / transformées
│   └── external/                # Données provenant de sources externes
│
├── tests/                       # Tests unitaires et d'intégration
│   ├── __init__.py
│   └── test_main.py
│
├── notebooks/                   # Notebooks Jupyter d'exploration
│   └── 01_exploration.ipynb
│
├── docs/                        # Documentation du projet
│
└── .gitlab/                     # Configuration CI/CD GitLab
                                 # (ou azure-pipelines.yml pour Azure DevOps)
```

---

## Fichiers de configuration

### `pyproject.toml`

Fichier central du projet Python. Il remplace les anciens `setup.py`, `setup.cfg` et `requirements.txt` en un seul fichier standardisé ([PEP 621](https://peps.python.org/pep-0621/)).

**Contenu type :**

```toml
[project]
name = "mon-projet"
version = "0.1.0"
description = "Description du projet"
authors = [
    { name = "Prénom Nom", email = "utilisateur@example.com" }
]
requires-python = ">=3.11"
dependencies = [
    "pandas>=2.0",
    "scikit-learn>=1.3",
]

[project.optional-dependencies]
dev = [
    "ruff",
    "pytest",
    "pre-commit",
    "interrogate",
]

[tool.ruff]
line-length = 120
select = ["E", "F", "I", "W"]

[tool.interrogate]
ignore-init-method = true
fail-under = 100

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.backends"
```

**Rôle des sections :**

| Section | Description |
|---|---|
| `[project]` | Nom, version, auteurs, dépendances du projet |
| `[project.optional-dependencies]` | Dépendances de développement (linting, tests) |
| `[tool.ruff]` | Configuration du linter/formatter Ruff |
| `[tool.interrogate]` | Seuil de couverture des docstrings |
| `[build-system]` | Système de build utilisé |

---

### `uv.lock`

Fichier généré automatiquement par `uv`. Il verrouille les versions exactes de toutes les dépendances (directes et transitives) pour garantir la reproductibilité de l'environnement.

**Bonnes pratiques :**
- Ne jamais modifier ce fichier manuellement
- Le versionner dans Git (il garantit que tous les développeurs utilisent les mêmes versions)
- Le regénérer avec `uv lock` après modification du `pyproject.toml`
- Installer l'environnement avec `uv sync`

---

### `.python-version`

Fichier contenant la version de Python utilisée par le projet. Utilisé par `uv` et d'autres outils pour sélectionner automatiquement l'interpréteur.

**Contenu type :**

```
3.11
```

---

### `.gitignore`

Fichier indiquant à Git les fichiers et dossiers à exclure du versionnement. Cela évite de committer des fichiers temporaires, des données volumineuses ou des informations sensibles.

**Contenu type :**

```gitignore
# Environnement virtuel
.venv/

# Cache Python
__pycache__/
*.pyc
*.pyo

# Variables d'environnement (contiennent des secrets)
.env

# Données (trop volumineuses pour Git)
data/raw/
data/processed/
data/external/
*.csv
*.parquet
*.xlsx

# Notebooks checkpoints
.ipynb_checkpoints/

# IDE
.vscode/
.idea/
*.swp

# Build
dist/
build/
*.egg-info/

# OS
.DS_Store
Thumbs.db
```

**Bonnes pratiques :**
- Ne jamais versionner de données brutes volumineuses (utiliser DVC, Git LFS ou un stockage cloud)
- Ne jamais versionner le fichier `.env` (contient des secrets : clés API, mots de passe)
- Toujours versionner le `.env.example` comme template
- Configurer les settings de votre IDE (VS Code, Cursor ou autre) en renseignant les fichiers comme "__pycache__" dans les **Exclude files** pour éviter de surcharger votre explorateur. Attention : cela ne vous libère pas du fichier **.gitignore**.

---

### `.pre-commit-config.yaml`

Configuration des hooks pre-commit. Ces hooks s'exécutent automatiquement avant chaque `git commit` pour vérifier la qualité du code.

**Contenu type :**

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: ['--maxkb=50000']
      - id: detect-private-key
      - id: no-commit-to-branch
        args: ['--branch', 'main']

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/econchick/interrogate
    rev: 1.7.0
    hooks:
      - id: interrogate
```

Pre-commit fournit plusieurs garde-fous essentiels :

- **Empêcher le commit de clés privées** (`detect-private-key`)
- **Empêcher les commits sur `main`** (`no-commit-to-branch`) — on travaille toujours sur une branche
- **Empêcher le commit de fichiers volumineux** (`check-added-large-files`, seuil par défaut : 50 000 KB)
- **Assurer la qualité du code** via Ruff (lint + formatage)
- **Vérifier la couverture des docstrings** via Interrogate

**Rôle des hooks courants :**

| Hook | Action |
|---|---|
| `trailing-whitespace` | Supprime les espaces en fin de ligne |
| `end-of-file-fixer` | Ajoute une ligne vide en fin de fichier |
| `check-yaml` | Valide la syntaxe des fichiers YAML |
| `check-added-large-files` | Bloque les fichiers de plus de 50 000 KB |
| `detect-private-key` | Bloque le commit si une clé privée est détectée |
| `no-commit-to-branch` | Empêche les commits directs sur `main` |
| `ruff` | Analyse statique du code Python (lint) |
| `ruff-format` | Formatage automatique du code Python |
| `interrogate` | Vérifie la couverture des docstrings |

Pour installer les hooks : `uv run pre-commit install` (cf. [Configuration d'un projet](05 Routine Projet.md)).

---

### `.env.example`

Template des variables d'environnement nécessaires au projet. Ce fichier est versionné et sert de documentation pour les nouveaux développeurs.

**Contenu type :**

```env
# Base de données
DATABASE_URL=postgresql://user:password@localhost:5432/dbname

# API
API_KEY=votre_cle_api
API_SECRET=votre_secret

# GCP
GOOGLE_APPLICATION_CREDENTIALS=path/to/credentials.json
PROJECT_ID=mon-projet-gcp
```

**Utilisation :**
1. Copier le fichier : `cp .env.example .env`
2. Remplir les valeurs réelles dans `.env`
3. Le fichier `.env` est exclu de Git via `.gitignore`

---

### `Makefile`

Le `Makefile` est un fichier de commandes raccourcies qui automatise les tâches répétitives du projet. Plutôt que de retaper des commandes longues ou de les chercher dans la documentation, `make <cible>` exécute la séquence associée en une frappe.

#### Syntaxe de base

Un Makefile est composé de **cibles** (*targets*). Chaque cible suit cette structure :

```makefile
cible: dépendances
	commande        # ← tabulation obligatoire (pas des espaces)
	commande
```

La **tabulation** en début de ligne est obligatoire — les espaces provoquent une erreur. Les éditeurs modernes (VS Code, Cursor) insèrent automatiquement une tabulation dans les fichiers `.makefile`.

La directive `.PHONY` indique à `make` que ces cibles ne correspondent pas à des fichiers — sans elle, `make` refuse d'exécuter une cible si un fichier du même nom existe déjà dans le dossier :

```makefile
.PHONY: install test lint format clean
```

#### Variables

Les variables Makefile évitent de dupliquer les chemins ou commandes répétées :

```makefile
PYTHON = uv run python
SRC    = src/
TESTS  = tests/

test:
	uv run pytest $(TESTS)

run:
	$(PYTHON) $(SRC)mon_projet/main.py
```

#### Dépendances entre cibles

Une cible peut dépendre d'une autre — `make ci` exécutera `lint` puis `test` automatiquement :

```makefile
ci: lint test
	@echo "✅ CI passée"
```

#### Makefile type pour un projet Python

```makefile
.PHONY: install test lint format lint-format interrogate coverage pre-commit-update clean ci

# ── Environnement ─────────────────────────────────────────────────────────────

install:
	uv sync

# ── Qualité du code ───────────────────────────────────────────────────────────

lint:
	uv run ruff check .

format:
	uv run ruff format .

lint-format:
	uv run ruff check . --fix
	uv run ruff format .

interrogate:
	uv run interrogate src/

# ── Tests ─────────────────────────────────────────────────────────────────────

test:
	uv run pytest

coverage:
	uv run pytest --cov=src --cov-report=term-missing --cov-report=html
	@echo "Rapport disponible dans htmlcov/index.html"

# ── Pipeline complète (utilisée en CI) ───────────────────────────────────────

ci: lint test

# ── Maintenance ───────────────────────────────────────────────────────────────

pre-commit-update:
	uv run pre-commit autoupdate

clean:
	find . -type d -name __pycache__ -exec rm -rf {} +
	find . -type d -name .ipynb_checkpoints -exec rm -rf {} +
	find . -type d -name .pytest_cache -exec rm -rf {} +
	find . -type d -name htmlcov -exec rm -rf {} +
	find . -name "*.pyc" -delete
```

**Usage :**

| Commande | Action |
|----------|--------|
| `make install` | Installe les dépendances du projet |
| `make lint` | Vérifie le code avec Ruff |
| `make format` | Formate le code avec Ruff |
| `make lint-format` | Lint + format en une seule commande |
| `make test` | Lance les tests avec pytest |
| `make coverage` | Tests + rapport de couverture HTML |
| `make ci` | Lint + tests (reproduit la CI en local) |
| `make interrogate` | Vérifie la couverture des docstrings |
| `make pre-commit-update` | Met à jour les hooks pre-commit |
| `make clean` | Supprime les fichiers temporaires |

> **Astuce** : lancer `make ci` avant chaque push permet de s'assurer que la CI passera, sans attendre le retour du pipeline distant.

---

### `LICENSE` et `CONTRIBUTING.md` — deux fichiers à ne pas confondre

Ces deux fichiers se trouvent à la racine de la plupart des projets, mais ils ont des rôles fondamentalement différents. Ils sont souvent confondus par les développeurs débutants, car tous deux sont des fichiers texte « administratifs » qui ne contiennent pas de code. Pourtant, ils répondent à deux questions bien distinctes :

- **`LICENSE`** répond à : *« Ai-je le droit d'utiliser, modifier ou redistribuer ce code ? »*
- **`CONTRIBUTING.md`** répond à : *« Comment puis-je participer à ce projet ? »*

Autrement dit, `LICENSE` est un document **juridique** qui définit ce qu'on **peut** faire avec le code, tandis que `CONTRIBUTING.md` est un document **organisationnel** qui explique comment on **doit** travailler sur le projet.

---

#### `LICENSE` — les droits légaux

Le fichier `LICENSE` définit les droits légaux associés au code source : qui peut l'utiliser, le copier, le modifier et le redistribuer, et sous quelles conditions. C'est un document juridique qui protège à la fois l'auteur (la personne ou l'entreprise qui a écrit le code) et les utilisateurs (ceux qui vont l'utiliser dans leurs propres projets).

**Pourquoi c'est important :** sans fichier `LICENSE`, le code est par défaut sous copyright exclusif de son auteur. La portée de cette règle dépend du contexte :

- **Pour un projet public** (open source, publié sur GitHub ou GitLab public) : sans licence, personne n'a légalement le droit de copier, modifier ou redistribuer le code, même s'il est visible par tous. Un projet public sans licence est donc un projet que personne ne peut réutiliser.
- **Pour un projet interne** (en entreprise) : le code est produit dans le cadre d'un contrat de travail. Il appartient à l'entreprise, et les employés ont implicitement le droit de l'utiliser dans le cadre de leurs missions. Un fichier `LICENSE` reste néanmoins recommandé pour formaliser les choses, notamment en cas de départ d'un collaborateur ou de partage du code avec un prestataire externe.

Voici un exemple typique :

```
Copyright (c) 2025 Mon Entreprise. All rights reserved.

This software is proprietary and confidential.
Unauthorized copying, distribution, or use of this software,
via any medium, is strictly prohibited.
```

Ce texte interdit à quiconque en dehors de l'entreprise de copier, distribuer ou utiliser le code. C'est la licence par défaut pour tout projet interne.

**Pour les projets open source** (projets destinés à être partagés publiquement), on utilise une licence standardisée reconnue par la communauté. Les trois plus courantes sont :

| Licence | Ce qu'elle autorise | Ce qu'elle impose | Cas d'usage |
|---------|--------------------|--------------------|-------------|
| **MIT** | Utilisation, modification, redistribution, usage commercial | Conserver la mention de copyright et la licence dans les copies | Projets simples, bibliothèques, outils — la plus utilisée |
| **Apache 2.0** | Mêmes droits que MIT | Conserver la licence + protection explicite contre les revendications de brevets | Projets d'entreprise, frameworks |
| **GPL v3** | Mêmes droits que MIT | Tout projet dérivé doit aussi être publié sous GPL (*copyleft*) | Projets qui veulent rester libres à jamais |

La différence clé entre ces licences réside dans la notion de **copyleft** : la GPL impose que tout projet qui réutilise du code GPL soit lui-même publié sous GPL, ce qui empêche de « fermer » le code. Les licences MIT et Apache n'ont pas cette contrainte : on peut intégrer du code MIT dans un projet propriétaire sans problème.

Voici à quoi ressemble une licence MIT complète :

```
MIT License

Copyright (c) 2025 Mon Entreprise

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

En résumé : on lit ce fichier pour savoir ce qu'on a le **droit** de faire avec le code. On ne le modifie quasiment jamais après la création du projet.

---

#### `CONTRIBUTING.md` — les règles de collaboration

Le fichier `CONTRIBUTING.md` est un guide destiné aux développeurs qui vont travailler sur le projet. Il n'a aucune valeur juridique : c'est un document pratique qui explique les conventions, le workflow et les attentes de l'équipe.

**Pourquoi c'est important :** sur un projet avec plusieurs développeurs, sans règles écrites, chacun organise son travail différemment. L'un crée des branches nommées `ma-feature`, l'autre `feature/ma-feature`, un troisième commit directement sur `main`. Le fichier `CONTRIBUTING.md` met tout le monde d'accord dès le départ et évite de répéter les mêmes consignes à chaque nouveau membre de l'équipe.

**Ce qu'on y trouve typiquement :**

1. **Le workflow de contribution** — comment créer une branche, développer, tester et soumettre ses modifications
2. **Les conventions de code** — style, nommage, format des commits
3. **Le processus de revue** — comment les modifications sont relues et validées

Les prérequis (outils à installer, commandes de setup) sont généralement documentés dans le `README.md` du projet. Le `CONTRIBUTING.md` n'a pas besoin de les répéter : il peut simplement renvoyer vers le README pour la partie installation, et se concentrer sur les règles de collaboration.

Voici un exemple complet :

```markdown
# Contribuer au projet

Ce document décrit les règles à suivre pour contribuer à ce projet.
Pour l'installation de l'environnement de développement, se référer au [README](README.md).

## Workflow de contribution

1. Créer une branche depuis `main` avec un préfixe décrivant le type de travail :

   | Préfixe | Usage | Exemple |
   |---------|-------|---------|
   | `feature/` | Nouvelle fonctionnalité | `feature/add-data-loader` |
   | `fix/` | Correction de bug | `fix/missing-null-check` |
   | `refactor/` | Restructuration du code sans changer le comportement | `refactor/simplify-pipeline` |
   | `docs/` | Ajout ou mise à jour de documentation | `docs/update-readme` |
   | `test/` | Ajout ou modification de tests | `test/add-loader-tests` |
   | `chore/` | Tâche technique (mise à jour de dépendances, CI, config) | `chore/upgrade-ruff` |
   | `hotfix/` | Correction urgente à déployer rapidement | `hotfix/fix-prod-crash` |
   | `data/` | Modification liée aux données ou aux pipelines de données | `data/add-preprocessing-step` |
   | `experiment/` | Exploration ou prototype (non destiné à être mergé tel quel) | `experiment/test-new-model` |

2. Développer et committer régulièrement avec des messages clairs, au format `<type>: <description>` :
   - En anglais, au mode impératif, en minuscules
   - Un commit par fichier modifié, petit et compréhensible
   - Types : `add`, `feat`, `fix`, `perf`, `ci`, `refacto`, `docs`, `test`, `revert`, `conf`
   - Exemples : `feat: add data loader`, `fix: null check in parser`
   - Pas de messages vagues comme `fix`, `update`, `wip`

3. Avant de pousser, vérifier :
   - Les tests passent : `uv run pytest`
   - Le linting passe : `uv run ruff check .`
   - Les docstrings sont présentes : `uv run interrogate .`

4. Pousser la branche et ouvrir une Merge Request (GitLab) ou Pull Request (GitHub)

5. Attendre la revue de code : au moins un autre développeur doit
   approuver la Merge Request / Pull Request avant qu'elle soit fusionnée dans `main`

## Conventions de code

- Style : PEP 8, vérifié automatiquement par Ruff
- Longueur de ligne : 120 caractères maximum
- Docstrings : format Google, obligatoires sur toutes les fonctions publiques
- Type hints : recommandés sur tous les arguments et valeurs de retour
```

En résumé : on lit ce fichier pour savoir comment **travailler** sur le projet. C'est un document vivant, mis à jour à mesure que les pratiques de l'équipe évoluent.

---

#### Récapitulatif `LICENSE` vs `CONTRIBUTING.md`

| | `LICENSE` | `CONTRIBUTING.md` |
|---|---|---|
| **Nature** | Document juridique | Document organisationnel |
| **Question** | *Ai-je le droit d'utiliser ce code ?* | *Comment contribuer à ce projet ?* |
| **Destiné à** | Toute personne qui accède au code | Les développeurs qui travaillent sur le projet |
| **Contenu** | Droits d'utilisation, de modification et de redistribution | Prérequis, workflow, conventions, processus de revue |
| **Fréquence de modification** | Quasiment jamais (défini une fois à la création) | Régulièrement (évolue avec les pratiques de l'équipe) |
| **Obligatoire ?** | Fortement recommandé (sans lui, personne ne peut légalement utiliser le code) | Recommandé dès que le projet a plus d'un contributeur |

---

## Dossiers du projet

### `src/`

Dossier contenant le code source principal du projet, organisé en package Python.

**Structure type :**

```
src/
└── mon_projet/
    ├── __init__.py          # Marque le dossier comme package Python
    ├── main.py              # Point d'entrée principal
    ├── config.py            # Chargement de la configuration
    ├── utils.py             # Fonctions utilitaires partagées
    ├── data/
    │   ├── __init__.py
    │   ├── loader.py        # Chargement des données
    │   └── preprocessor.py  # Nettoyage et transformation
    ├── models/
    │   ├── __init__.py
    │   ├── train.py         # Entraînement des modèles
    │   └── predict.py       # Inférence / prédiction
    └── visualization/
        ├── __init__.py
        └── plots.py         # Génération de graphiques
```

**Bonnes pratiques :**
- Séparer clairement les responsabilités (chargement, traitement, modélisation, visualisation)
- Chaque module doit avoir un rôle précis et cohérent
- Le code réutilisable va dans `src/`, le code exploratoire dans `notebooks/`

---

### `data/`

Dossier contenant les données du projet, organisé en sous-dossiers selon le niveau de traitement.

```
data/
├── raw/           # Données brutes, jamais modifiées
├── processed/     # Données nettoyées, prêtes pour la modélisation
└── external/      # Données provenant de sources tierces
```

**Bonnes pratiques :**
- Les données brutes (`raw/`) ne doivent jamais être modifiées manuellement
- Le passage de `raw/` à `processed/` doit être reproductible via un script
- Ne pas versionner les fichiers volumineux dans Git (utiliser `.gitignore`)
- Pour les fichiers volumineux, privilégier un stockage externe (GCS, S3, DVC)

---

### `tests/`

Dossier contenant les tests automatisés du projet, exécutés avec `pytest`.

**Structure type :**

```
tests/
├── __init__.py
├── test_main.py           # Tests du module principal
├── test_utils.py          # Tests des fonctions utilitaires
├── test_loader.py         # Tests du chargement de données
└── conftest.py            # Fixtures partagées entre les tests
```

**Bonnes pratiques :**
- Nommer les fichiers de test `test_<module>.py` (convention pytest)
- Écrire au minimum des tests pour les fonctions critiques (transformations de données, logique métier)
- Utiliser des fixtures (`conftest.py`) pour les données de test réutilisables
- Exécuter les tests avec `uv run pytest`

---

### `notebooks/`

Dossier contenant les notebooks Jupyter utilisés pour l'exploration, l'analyse et le prototypage.

**Convention de nommage :**

```
notebooks/
├── 01_exploration.ipynb
├── 02_feature_engineering.ipynb
├── 03_modelisation.ipynb
└── 04_evaluation.ipynb
```

**Bonnes pratiques :**
- Numéroter les notebooks pour indiquer l'ordre d'exécution
- Les notebooks servent à l'exploration ; le code validé doit être déplacé dans `src/`
- Nettoyer les outputs avant de committer (évite les conflits Git et réduit la taille du dépôt)

---

### `docs/`

Dossier destiné à la documentation complémentaire du projet : spécifications techniques, schémas d'architecture, notes de réunion, documentation API.

---

### `.gitlab/` / `.azuredevops/`

Dossier de configuration spécifique à la plateforme CI/CD utilisée.

**GitLab** : le dossier `.gitlab/` peut contenir des templates de merge request, des fichiers de CI/CD, ou des redirections. Le fichier principal de pipeline est `.gitlab-ci.yml` situé à la racine du projet.

**Azure DevOps** : le fichier principal de pipeline est `azure-pipelines.yml` situé à la racine du projet. Les pipelines supplémentaires ou templates peuvent être organisés dans un dossier `.azuredevops/` ou `pipelines/`.

| Plateforme | Fichier principal | Dossier de configuration |
|---|---|---|
| GitLab | `.gitlab-ci.yml` | `.gitlab/` |
| Azure DevOps | `azure-pipelines.yml` | `.azuredevops/` ou `pipelines/` |

---

### `Dockerfile` et `.dockerignore`

Ces deux fichiers sont présents lorsque le projet est conteneurisé avec Docker.

**`Dockerfile`** — fichier d'instructions pour construire l'image Docker du projet. Il décrit l'environnement d'exécution : système de base, dépendances installées, fichiers copiés et commande de lancement.

**Contenu type :**

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --no-dev

COPY src/ ./src/

CMD ["python", "-m", "mon_projet.main"]
```

**`.dockerignore`** — fichier indiquant à Docker les fichiers et dossiers à exclure lors de la construction de l'image. Son rôle est analogue à `.gitignore` mais pour Docker. Il permet de réduire la taille de l'image et d'éviter d'y inclure des fichiers inutiles ou sensibles.

**Contenu type :**

```
.git/
.venv/
__pycache__/
*.pyc
data/
notebooks/
tests/
docs/
.env
*.md
```

**Bonnes pratiques :**
- Toujours créer un `.dockerignore` aux côtés du `Dockerfile` pour éviter de copier des fichiers inutiles dans l'image
- Placer les instructions qui changent le moins souvent en haut du `Dockerfile` (pour optimiser le cache Docker)
- Ne jamais inclure de secrets (`.env`, clés) dans l'image Docker
