# Cas 1 — Créer un nouveau projet

← [Retour au sommaire](05 Routine Projet.md)

---

Ce cas de figure se présente quand on démarre un projet de zéro. Deux approches possibles : le **Cookiecutter Data Science** (recommandé, structure standardisée) ou une **initialisation manuelle avec `uv`** (projets simples ou hors Cookiecutter).

---

## Option A — Avec le Cookiecutter Data Science (recommandé)

On utilise le **[Cookiecutter Data Science](https://github.com/drivendataorg/cookiecutter-data-science)** pour générer une structure de projet standardisée (cf. [Structure d'un projet](03 Structure Projet.md) pour le détail de chaque fichier et dossier).

Dépôt GitHub : https://github.com/drivendataorg/cookiecutter-data-science

---

## Étape 1 — Générer le projet

Dans Git Bash :

```bash
pip install cookiecutter
```

Puis :

```bash
cd Desktop

cookiecutter https://github.com/drivendataorg/cookiecutter-data-science
```

Il faut alors renseigner :

* project_name
* repository_name (optionnel, on peut passer avec enter si le nom entre parenthèse nous convient)
* author_name
* description
* license

---

## Étape 2 — Installer l'environnement

Une fois le projet généré, l'ouvrir dans VS Code puis exécuter dans le terminal :

```bash
# 1. Fixer la version de Python du projet
uv python pin 3.12                    # crée le fichier .python-version

# 2. Créer et activer l'environnement virtuel
uv venv
source .venv/Scripts/activate         # Git Bash
# ou : .venv\Scripts\activate         # CMD
# ou : .venv\Scripts\Activate.ps1     # PowerShell

# 3. Installer les dépendances du projet
uv sync

# 4. Ajouter les outils de développement
uv add --dev setuptools
uv add --dev pre-commit
uv add --dev pytest
uv add --dev ruff
uv add --dev interrogate
uv sync

# 5. Installer les hooks pre-commit
uv run pre-commit install
```

Le projet est prêt. On peut commencer à coder.

---

## Étape 3 — Activer le Wiki et le CI/CD

Avant de commencer à coder, vérifier que le dépôt distant dispose bien d'un **wiki** et d'un **pipeline CI/CD**. Sur un nouveau projet, ces options ne sont pas toujours activées par défaut.

### Wiki

Le wiki centralise la documentation du projet (architecture, conventions, décisions techniques…).

- **GitHub / GitLab** : le wiki n'est pas activé par défaut. Aller dans **Settings** (ou **Paramètres**) du dépôt et activer l'option Wiki.
- **Azure DevOps** : un espace Wiki est disponible d'office dans chaque projet, accessible depuis le menu latéral. Rien à activer.

### Pipeline CI/CD

Un pipeline CI/CD automatise les tests, la vérification de la qualité du code et le déploiement.

- **GitHub** : activer **GitHub Actions** dans **Settings → Actions** et créer un workflow dans le dossier `.github/workflows/`.
- **GitLab** : activer les pipelines dans **Settings → CI/CD** et créer un fichier `.gitlab-ci.yml` à la racine du dépôt.
- **Azure DevOps** : créer un pipeline depuis **Pipelines → New Pipeline** et créer un fichier `azure-pipeline.yaml` à la racine du dépôt.

Ces deux éléments doivent être mis en place dès la création du projet.

---

## Option B — Sans Cookiecutter (initialisation manuelle avec `uv`)

Pour les projets simples, on peut initialiser un projet manuellement.

### Étape 1 — Initialiser le projet

```bash
# Initialiser le projet avec une version Python spécifique
uv init --python 3.12

# (optionnel) Changer la version après l'init
uv python pin 3.13
```

### Étape 2 — Ajouter les dépendances

```bash
# Dépendance principale
uv add <package>

# Dépendance de dev
uv add --dev <package>

# Exemple courant
uv add --dev pytest ruff mypy
```

### Étape 3 — Installer et configurer pre-commit

```bash
# Ajouter pre-commit comme dépendance de dev
uv add --dev pre-commit

# Initialiser git et installer les hooks
git init
uv run pre-commit install
```

Créer un fichier `.pre-commit-config.yaml` à la racine (cf. [Pre-commit : maîtriser ses hooks](06 Pre-commit.md) pour le détail complet) :

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.11.13
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
```

```bash
# Tester les hooks sur tous les fichiers
uv run pre-commit run --all-files
```

### Étape 4 — Lancer des commandes dans le venv

```bash
# Exécuter un script
uv run python mon_script.py

# Lancer les tests
uv run pytest

# Lancer ruff
uv run ruff check .
```

### Précision : Supprimer le venv et repartir de zéro si besoin

```bash
# Option 1 : suppression simple du venv
rm -rf .venv

# Option 2 : suppression venv + lock file (reset complet)
rm -rf .venv uv.lock

# Recréer tout
uv sync

# Réinstaller les hooks pre-commit
uv run pre-commit install
```

### Résumé one-liner : projet from scratch

```bash
uv init --python 3.12 && uv add --dev pre-commit pytest ruff && git init && uv run pre-commit install
```
