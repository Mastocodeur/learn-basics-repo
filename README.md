# Wiki Technique

Bienvenue sur le wiki technique. Ce wiki accompagne les nouveaux arrivants dans leur prise en main de l'environnement de développement Python et des pratiques en vigueur dans une équipe Data. Il sert également de référence au quotidien pour l'ensemble des collaborateurs.

---

## Chapitres

### 01 — Guide d'installation

Toutes les étapes pour configurer un poste de travail : installation de Python, VS Code, Cursor, Git, uv, MSYS2, Node.js, configuration de Git et des extensions VS Code recommandées.

### 02 — Conventions de code Python

Les règles de style et de nommage à respecter dans les projets Python, basées sur PEP 8. Couvre les conventions de nommage, l'indentation, les imports, les commentaires et docstrings (format Google), la longueur de ligne, les type hints et les bonnes pratiques générales.

### 03 — Structure d'un projet

Description de la structure type d'un projet Python data science généré par le [Cookiecutter Data Science](https://github.com/drivendataorg/cookiecutter-data-science). Détaille le rôle de chaque fichier de configuration (`pyproject.toml`, `uv.lock`, `.pre-commit-config.yaml`, `Makefile`…) et de chaque dossier (`src/`, `data/`, `tests/`, `notebooks/`, `docs/`…).

### 04 — Formats de fichiers

Présentation des formats de fichiers courants dans un projet data : YAML, JSON, TOML, Markdown, `.env`, CSV et Parquet. Pour chaque format : syntaxe, cas d'usage et exemples de lecture/écriture en Python.

### 05 — Configuration d'un projet

Guide opérationnel pour démarrer ou rejoindre un projet. Découpé en sous-parties :

- **05a — La base : `uv`** — Comprendre `uv`, les environnements virtuels, les commandes essentielles et le workflow quotidien.
- **05b — Créer un nouveau projet** — Générer un projet from scratch avec le Cookiecutter Data Science, initialiser Git et configurer l'environnement.
- **05c — Rejoindre un projet existant** — Cloner un dépôt, créer une branche et installer l'environnement selon ce qu'on trouve (`pyproject.toml`, `requirements.txt` ou rien).
- **05d — Exercice pratique : outils qualité** — Un code volontairement mauvais pour découvrir pre-commit, Ruff, pytest et interrogate en conditions réelles.

### 06 — Pre-commit : maîtriser ses hooks

Guide de référence complet sur pre-commit. Couvre le principe de fonctionnement, le flux d'exécution, la structure du fichier de configuration, le catalogue des hooks (génériques, Python, autres langages), le contrôle du périmètre, les stages, les hooks locaux, les commandes essentielles et la config recommandée pour un projet Python.
