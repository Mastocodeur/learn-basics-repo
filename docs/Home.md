# Wiki Technique

Ce wiki rassemble les bonnes pratiques, outils et conventions recommandés pour travailler efficacement sur des projets Python. Il couvre l'ensemble du cycle de vie d'un projet : configuration du poste, structure du code, gestion des dépendances, qualité et outillage.

L'objectif est de centraliser en un seul endroit tout ce qu'il faut savoir pour démarrer rapidement et travailler de manière cohérente, que l'on soit seul ou en équipe.

Ce wiki est **vivant** : il évolue avec les pratiques et les outils. 

Toute contribution est la bienvenue — voir [CONTRIBUTING.md](CONTRIBUTING.md).

Bonne lecture à tous !
---

## Chapitres

### 01 — Guide d'installation

Toutes les étapes pour configurer un poste de travail : installation de Python, VS Code, Cursor, Git, uv, MSYS2, Node.js, configuration de Git et des extensions VS Code recommandées.

### 02 — Conventions de code Python

Les règles de style et de nommage à respecter dans les projets Python, basées sur PEP 8. Couvre les conventions de nommage, l'indentation, les imports, les commentaires et docstrings (format Google), la longueur de ligne, les type hints (types de base, unions, TypedDict, dataclasses, Protocol, Callable, mypy) et les bonnes pratiques générales.

### 03 — Structure d'un projet

Description de la structure type d'un projet Python data science généré par le [Cookiecutter Data Science](https://github.com/drivendataorg/cookiecutter-data-science). Détaille le rôle de chaque fichier de configuration (`pyproject.toml`, `uv.lock`, `.pre-commit-config.yaml`, `Makefile`…) et de chaque dossier (`src/`, `data/`, `tests/`, `notebooks/`, `docs/`…). La section Makefile couvre la syntaxe, les variables, les dépendances entre cibles et un Makefile type complet.

### 04 — Formats de fichiers

Présentation des formats de fichiers courants dans un projet data : YAML, JSON, TOML, Markdown, `.env`, CSV et Parquet. Pour chaque format : syntaxe, cas d'usage et exemples de lecture/écriture en Python.

### 05 — Configuration d'un projet

Guide opérationnel pour démarrer ou rejoindre un projet. Découpé en sous-parties :

- **05a — La base : `uv`** — Comprendre `uv`, les environnements virtuels, les commandes essentielles et le workflow quotidien.
- **05b — Créer un nouveau projet** — Générer un projet from scratch avec le Cookiecutter Data Science, initialiser Git et configurer l'environnement.
- **05c — Rejoindre un projet existant** — Cloner un dépôt, créer une branche et installer l'environnement selon ce qu'on trouve (`pyproject.toml`, `requirements.txt` ou rien).

### 06 — Pre-commit : maîtriser ses hooks

Guide de référence complet sur pre-commit. Couvre le principe de fonctionnement, le flux d'exécution, la structure du fichier de configuration, le catalogue des hooks (génériques, Python, autres langages), le contrôle du périmètre, les stages, les hooks locaux, les commandes essentielles et la config recommandée pour un projet Python.

### 07 — Git au quotidien

Les pratiques Git du quotidien : conventions de commits (Conventional Commits), workflow de branches, rebase vs merge, gestion des conflits et commandes utiles.

### 08 — Docker

Introduction à Docker pour les projets Python : Dockerfile, commandes essentielles, Docker Compose, bonnes pratiques et cas d'usage data / ML.

### 09 — Variables d'environnement et secrets

Comment gérer proprement la configuration et les secrets : fichiers `.env` et `.env.example`, lecture en Python, ce qu'on ne commit jamais, et partage sécurisé en équipe.

### 10 — CI/CD

Automatiser les tests et le déploiement avec GitHub Actions, GitLab CI et Azure Pipelines. Exemples de workflows complets, gestion des secrets dans les pipelines et bonnes pratiques.

### 11 — Tests avec pytest

Écrire et organiser des tests automatisés avec pytest. Couvre les fixtures, le parametrize, le mocking avec pytest-mock, la mesure de coverage et la configuration dans `pyproject.toml`.

### 12 — Logging

Tracer le comportement d'une application avec le module `logging` de la bibliothèque standard et avec Loguru. Couvre les niveaux de log, la configuration par variables d'environnement, la rotation de fichiers et les bonnes pratiques (ne jamais logger de secrets, contextualiser les messages).
