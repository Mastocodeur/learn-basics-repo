# La base : `uv`

← [Retour au sommaire](05 Routine Projet.md)

---

## Sommaire

- [Qu'est-ce que `uv` ?](#quest-ce-que-uv-)
- [Environnement virtuel](#environnement-virtuel)
- [Commandes essentielles](#commandes-essentielles)
- [Dépendances vs dépendances de développement](#dépendances-vs-dépendances-de-développement)
- [Exemple de workflow quotidien](#exemple-de-workflow-quotidien)
- [Commandes utiles](#commandes-utiles)

---

## Qu'est-ce que `uv` ?

[uv](https://docs.astral.sh/uv/) est un gestionnaire de projet Python développé par [Astral](https://astral.sh/) (la même équipe que Ruff). Écrit en Rust, il est **10 à 100 fois plus rapide** que `pip` et remplace à lui seul plusieurs outils historiques.

> **Rust, c'est quoi ?** Rust est un langage de programmation compilé, conçu pour être très rapide et économe en mémoire (comparable au C/C++). De plus en plus d'outils Python sont réécrits en Rust pour gagner en performance : `uv`, `ruff`, `polars`, `pydantic`… En tant qu'utilisateur, on n'a pas besoin de connaître Rust — on profite simplement de la vitesse.

Les outils remplacés par `uv` :

| Outil remplacé | Rôle | Remplacé par |
|---|---|---|
| `pip` | Installation de packages | `uv add`, `uv sync` |
| `pip-tools` (`pip-compile`) | Verrouillage des versions | `uv lock` |
| `virtualenv` / `venv` | Création d'environnement virtuel | Géré automatiquement par `uv` |
| `pyenv` | Gestion des versions de Python | `uv python install` |

En résumé : **un seul outil pour tout**. Plus besoin de jongler entre `pip install`, `python -m venv`, `pip freeze`, etc.

---

## Environnement virtuel

Un environnement virtuel est un dossier isolé (`.venv/`) qui contient sa propre copie de Python et des packages installés. Il permet à chaque projet d'avoir ses propres dépendances sans interférer avec les autres projets ou avec le Python installé globalement sur la machine.

Sans environnement virtuel, tous les projets partagent les mêmes packages. Si le projet A a besoin de `pandas 1.5` et le projet B de `pandas 2.1`, l'un des deux cassera. L'environnement virtuel résout ce problème en isolant chaque projet.

Avec `uv`, l'environnement virtuel est stocké dans `.venv/` à la racine du projet.

**L'environnement ne se crée qu'une seule fois.** La commande `uv venv` génère le dossier `.venv/` qui persiste sur le disque. Les fois suivantes, il suffit de l'activer — pas besoin de le recréer. Concrètement :

**Première fois** (création + activation) :

```bash
uv venv                              # crée le dossier .venv/ (une seule fois)
source .venv/Scripts/activate         # Git Bash
# ou : .venv\Scripts\activate         # CMD
# ou : .venv\Scripts\Activate.ps1     # PowerShell
```

**Les fois suivantes** (activation seulement) :

```bash
source .venv/Scripts/activate         # Git Bash
# ou : .venv\Scripts\activate         # CMD
# ou : .venv\Scripts\Activate.ps1     # PowerShell
```

Une fois l'environnement activé, le terminal affiche `(.venv)` au début de la ligne. Toutes les commandes `uv sync`, `uv add`, `uv run` s'exécutent alors dans cet environnement isolé. Quand on ferme le terminal, l'environnement se désactive automatiquement — il faudra le réactiver à la prochaine ouverture.

> **Problème courant : l'IDE bloque l'activation.** Sur Windows, PowerShell peut refuser d'exécuter le script d'activation avec une erreur de type *"execution of scripts is disabled on this system"*. C'est une restriction de la politique d'exécution de PowerShell. Pour la débloquer, exécuter cette commande **une seule fois** dans PowerShell :
>
> ```powershell
> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
> ```
>
> Après cela, l'activation fonctionnera normalement. Si le problème persiste, utiliser Git Bash ou CMD à la place de PowerShell.

---

## Commandes essentielles

Le tableau ci-dessous regroupe les commandes `uv` utilisées au quotidien. Pas besoin de toutes les retenir : `uv sync` et `uv add` couvrent 90% des besoins.

| Commande | Ce qu'elle fait | Quand l'utiliser |
|---|---|---|
| `uv init` | Initialise un nouveau projet (crée `pyproject.toml`) | Au démarrage d'un projet sans Cookiecutter |
| `uv add <package>` | Ajoute une dépendance au projet | Quand on a besoin d'un nouveau package |
| `uv add --dev <package>` | Ajoute une dépendance de développement | Pour les outils qui ne vont pas en production (ruff, pytest…) |
| `uv remove <package>` | Supprime une dépendance | Quand un package n'est plus utilisé |
| `uv sync` | Installe toutes les dépendances et crée le `.venv/` | En arrivant sur un projet, ou après un `git pull` |
| `uv lock` | Regénère le fichier `uv.lock` | Après modification manuelle du `pyproject.toml` |
| `uv run <commande>` | Exécute une commande dans l'environnement du projet | Pour lancer un script, des tests, un outil |
| `uv python install 3.12` | Installe une version de Python | Quand le projet requiert une version qu'on n'a pas |
| `uv python pin 3.12` | Crée le fichier `.python-version` avec la version choisie | Au démarrage d'un projet, pour fixer la version Python |
| `uv pip freeze > requirements.txt` | Génère un `requirements.txt` depuis l'environnement actuel | Quand un service externe exige un `requirements.txt` (Docker, Cloud Functions…) |

---

## Dépendances vs dépendances de développement

`uv` distingue deux catégories de dépendances, déclarées dans des sections différentes du `pyproject.toml` :

**Dépendances du projet** (`uv add <package>`) — les packages nécessaires au fonctionnement du code. Ce sont les bibliothèques que le code importe directement : `pandas`, `scikit-learn`, `requests`, etc. Elles sont installées partout, y compris en production.

```toml
[project]
dependencies = [
    "pandas>=2.0",
    "scikit-learn>=1.3",
]
```

**Dépendances de développement** (`uv add --dev <package>`) — les outils qui servent uniquement pendant le développement : linter, formatter, tests, vérification de docstrings. Ils ne sont pas nécessaires en production et ne sont pas inclus dans l'image Docker ou le déploiement.

```toml
[project.optional-dependencies]
dev = [
    "ruff",
    "pytest",
    "pre-commit",
    "interrogate",
]
```

La distinction est importante : en production, on installe uniquement les dépendances du projet (avec `uv sync --no-dev`) pour obtenir une image plus légère et plus sécurisée.

---

## Exemple de workflow quotidien

```bash
# Arriver sur un projet existant
git clone git@github.com:mon-organisation/mon-projet.git
cd mon-projet

# Créer et activer l'environnement virtuel
uv venv
source .venv/Scripts/activate         # Git Bash
# ou : .venv\Scripts\activate         # CMD
# ou : .venv\Scripts\Activate.ps1     # PowerShell

# Installer les dépendances
uv sync

# Ajouter un nouveau package
uv add requests                       # dépendance du projet
uv add --dev pytest-cov               # outil de dev

# Lancer des commandes dans l'environnement du projet
uv run python src/mon_projet/main.py  # exécuter un script
uv run pytest                         # lancer les tests
uv run ruff check .                   # vérifier le code

# Après un git pull (un collègue a ajouté des dépendances)
uv sync                               # met à jour l'environnement
```

---

## Commandes utiles

En complément des commandes `uv` du quotidien, voici quelques commandes pratiques pour initialiser ou maintenir un projet :

**Fixer la version Python du projet :**

```bash
uv python pin 3.12
```

Crée le fichier `.python-version` à la racine du projet. Ce fichier est lu par `uv` et les IDE pour sélectionner automatiquement la bonne version de Python. À faire au démarrage de tout nouveau projet.

**Générer un `requirements.txt` depuis le `pyproject.toml` :**

```bash
uv pip freeze > requirements.txt
```

Utile quand un service externe exige un `requirements.txt` (Dockerfile, Google Cloud Functions, AWS Lambda…). Le fichier est généré avec les versions exactes de l'environnement actuel.

**Générer un `.gitignore` complet :**

```bash
curl -L https://www.toptal.com/developers/gitignore/api/python,jupyter,macos,linux,windows > .gitignore
```

Cette commande interroge le service [gitignore.io](https://www.toptal.com/developers/gitignore) et génère un `.gitignore` adapté à un projet Python avec Jupyter, couvrant les trois systèmes d'exploitation (macOS, Linux, Windows). Le résultat est bien plus complet que ce qu'on écrirait à la main : il couvre les fichiers temporaires, les caches, les fichiers d'IDE, etc.
