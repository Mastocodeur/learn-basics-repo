# Cas 2 — Rejoindre un projet existant

← [Retour au sommaire](05 Routine Projet.md)

---

## Sommaire

- [Cloner le dépôt](#cloner-le-dépôt)
- [Créer une branche de travail](#créer-une-branche-de-travail)
- [Branches d'intégration et de déploiement](#branches-dintégration-et-de-déploiement)
- [Écrire des messages de commit](#écrire-des-messages-de-commit)
- [Tags et versions](#tags-et-versions)
- [Merge Requests](#merge-requests)
- [Issues](#issues)
- [Le projet a un `pyproject.toml`](#le-projet-a-un-pyprojecttoml)
- [Le projet a un `requirements.txt`](#le-projet-a-un-requirementstxt)
  - [Situation A — Versions spécifiées](#situation-a--versions-spécifiées)
  - [Situation B — Versions absentes ou vagues](#situation-b--versions-absentes-ou-vagues)
- [Le projet n'a pas de requirements](#le-projet-na-pas-de-requirements)

---

**Ce cas de figure est le plus fréquent** : on arrive sur un projet déjà en cours et on doit récupérer le code et installer l'environnement de développement.

## Cloner le dépôt

La première étape est toujours la même — cloner le dépôt :

```bash
git clone git@github.com:<organisation>/<nom-du-projet>.git
cd <nom-du-projet>
```

---

## Créer une branche de travail

**Règle fondamentale : on ne travaille jamais directement sur `main` ou sur `master`.** Avant toute modification, la première chose à faire est de créer une branche dédiée. Le nom de la branche doit refléter le type de travail et son contenu, en `kebab-case`.

```bash
git checkout -b <type>/<description-courte>
```

Les types de branches suivent une convention qui permet de comprendre immédiatement la nature du travail :

| Type | Usage | Quand l'utiliser | Exemple |
|------|-------|------------------|---------|
| `feature/` | Nouvelle fonctionnalité | On ajoute un comportement qui n'existait pas avant (nouvel endpoint, nouveau script, nouvelle page) | `feature/add-data-loader` |
| `fix/` | Correction de bug | Le code existant produit un résultat incorrect ou une erreur — on corrige sans changer la logique métier | `fix/missing-null-check` |
| `refactor/` | Restructuration du code | On réorganise ou simplifie du code existant sans changer son comportement visible (renommages, découpage de fonctions, suppression de code mort) | `refactor/simplify-pipeline` |
| `docs/` | Documentation | On modifie uniquement des fichiers de documentation (README, wiki, docstrings) sans toucher au code fonctionnel | `docs/update-readme` |
| `test/` | Tests | On ajoute, corrige ou complète des tests sans modifier le code de production | `test/add-loader-tests` |
| `chore/` | Tâche technique | On met à jour des dépendances, de la config CI, des outils de dev — aucun impact sur le code métier | `chore/upgrade-ruff` |
| `hotfix/` | Correction urgente | Un bug critique est en production et doit être corrigé immédiatement, sans attendre le cycle normal de review | `hotfix/fix-prod-crash` |
| `data/` | Données ou pipelines | On modifie un pipeline de données, un schéma, un script d'ingestion ou de preprocessing | `data/add-preprocessing-step` |
| `experiment/` | Exploration / prototype | On teste une idée, un modèle, une approche — la branche n'est pas destinée à être mergée telle quelle | `experiment/test-new-model` |

**Exemple concret :** on rejoint un projet et on doit ajouter un script de chargement de données :

```bash
git checkout -b feature/add-data-loader
```

Une fois la branche créée, on peut passer à l'installation de l'environnement. Ensuite, la marche à suivre dépend de ce qu'on trouve dans le dépôt. Trois cas de figure se présentent.


---

## Branches d'intégration et de déploiement

Les branches listées ci-dessus (`feature/`, `fix/`…) sont des **branches de travail** — elles ont une durée de vie courte et sont créées par les développeurs pour un changement précis. Elles s'intègrent dans une hiérarchie de branches plus stables, dont le rôle est de gérer l'intégration et le déploiement.

### La hiérarchie des branches

```
feature/ma-fonctionnalite  ──┐
fix/mon-bug                  ├──► dev ──► main
refactor/mon-refacto        ──┘
```

| Branche | Rôle | Qui y touche |
|---------|------|--------------|
| `feature/*`, `fix/*`… | Travail en cours — courte durée | Chaque développeur sur sa propre branche |
| `dev` | Branche d'intégration — reçoit toutes les PR de travail | Personne ne commit directement dessus |
| `main` | Branche de production — code stable et déployé | Alimentée uniquement depuis `dev` via PR |

**Règle fondamentale :** on ne commit jamais directement sur `dev` ou `main`. Tout passe par une Pull Request (ou Merge Request).

### Le flux de travail complet

1. On crée une branche depuis `dev` : `git checkout -b feature/ma-fonctionnalite`
2. On travaille, on commit, on push
3. On ouvre une PR de `feature/ma-fonctionnalite` → `dev`
4. La PR est relue et mergée dans `dev`
5. Quand `dev` est stable et validé, une PR est ouverte de `dev` → `main`
6. Le merge dans `main` déclenche le déploiement en production via le pipeline CI/CD

### Les branches d'infrastructure

Certains projets ajoutent des branches dédiées à l'infrastructure de déploiement. Deux cas courants :

**Branche `aks`** (Azure Kubernetes Service)

Contient les manifests Kubernetes (fichiers YAML) qui décrivent comment déployer l'application sur un cluster AKS (le service Kubernetes managé d'Azure) : Deployment, Service, ConfigMap, Secret, Ingress… Cette branche est lue par le pipeline CI/CD pour savoir comment déployer quand `main` est mis à jour.

**Branche `gke`** (Google Kubernetes Engine)

Même principe, mais pour les clusters Google Cloud. Le contenu est identique dans sa structure (manifests Kubernetes), seules les spécificités liées à Google Cloud diffèrent (annotations, classes de stockage, load balancers GCP…).

> **Note :** certaines équipes font le choix de ne pas avoir de branche `aks` ou `gke` séparée et d'intégrer les fichiers d'infrastructure directement dans `dev` et `main`. C'est un choix valide qui simplifie la gestion des branches, particulièrement adapté aux petites équipes.

### Quand faire quoi

| Situation | Action |
|-----------|--------|
| Je commence une nouvelle tâche | `git checkout dev && git pull origin dev` puis `git checkout -b feature/ma-tache` |
| Ma PR est mergée dans `dev` | Supprimer la branche locale : `git branch -d feature/ma-tache` |
| Je veux récupérer les dernières modifs de `dev` | `git fetch origin && git rebase origin/dev` |
| `dev` est validé et prêt pour la prod | Ouvrir une PR `dev` → `main` |
| Un bug critique est en prod | Créer une `hotfix/` depuis `main`, corriger, PR → `main` puis merger `main` dans `dev` |

---

## Écrire des messages de commit

**Règle : un commit par changement cohérent, avec des messages courts et compréhensibles.** Éviter les commits volumineux qui regroupent plusieurs changements non liés — plus un commit est petit, plus il est facile à relire et à annuler si nécessaire.

Chaque message de commit suit le format :

```
<type>: <description>
```

Les types disponibles sont :

| Type | Usage | Quand l'utiliser | Exemple |
|------|-------|------------------|---------|
| `add` | Ajout de fichiers ou composants | On crée un nouveau fichier, module ou composant qui n'existait pas — sans logique métier complexe (un utilitaire, un script, un fichier de config) | `add: data loader module` |
| `feat` | Création d'une fonctionnalité | On implémente un comportement métier nouveau (un endpoint, un algorithme, une règle de gestion) — plus qu'un simple ajout de fichier | `feat: user authentication endpoint` |
| `fix` | Correction de bug | Le code existant produit un résultat incorrect ou une erreur — on corrige le comportement sans changer la logique | `fix: null check in data parser` |
| `perf` | Amélioration des performances | On optimise du code existant (requête SQL, algorithme, cache) sans changer son comportement fonctionnel | `perf: optimize batch processing query` |
| `ci` | Intégration continue | On modifie un fichier de pipeline CI/CD (`.github/workflows/`, `.gitlab-ci.yml`) ou la config de déploiement | `ci: add linting step to pipeline` |
| `refacto` | Refactorisation | On restructure du code existant sans changer son comportement ni corriger de bug (renommage, découpage, simplification) | `refacto: simplify data pipeline logic` |
| `docs` | Documentation | On modifie uniquement de la documentation (README, wiki, docstrings, commentaires) sans toucher au code fonctionnel | `docs: add best practices to README.md` |
| `test` | Tests | On ajoute, corrige ou complète des tests sans modifier le code de production | `test: add unit tests for data loader` |
| `revert` | Retour arrière | On annule un commit précédent qui a introduit un problème — on revient à l'état antérieur | `revert: undo breaking change in config` |
| `conf` | Configuration | On modifie un fichier de configuration du projet (`pyproject.toml`, `.pre-commit-config.yaml`, `Makefile`) sans impact sur le code métier | `conf: update ruff settings` |

**Bonnes pratiques :**

- Écrire en anglais, au mode impératif, en minuscules
- Un commit = un changement logique cohérent
- Éviter les messages vagues comme `fix`, `update`, `wip`

---

## Tags et versions

Les tags servent à marquer les versions stables du projet. Ils suivent le **versionnement sémantique** avec le format `vX.Y.Z` :

| Composant | Signification | Exemple |
|-----------|---------------|---------|
| `X` (majeur) | Changement majeur, potentiellement incompatible | `v2.0.0` |
| `Y` (mineur) | Nouvelle fonctionnalité, rétrocompatible | `v1.3.0` |
| `Z` (patch) | Correction de bug | `v1.3.1` |

**Créer un tag :**

```bash
git tag v1.0.0
git push origin v1.0.0
```

**Règle : chaque nouveau tag doit être accompagné d'une release** (sur GitLab : *Deploy → Releases*, sur GitHub : *Releases → Draft a new release*). La release décrit les changements inclus dans cette version.

---


## Durée de vie des branches

**Une branche qui traîne devient un problème.** Plus une branche vit longtemps, plus elle s'éloigne de `main` et plus le merge sera douloureux.

**Règle : une branche de travail ne devrait pas dépasser 3 à 5 jours.** Si c'est le cas, c'est en général le signe que le périmètre est trop large — il faut découper.

**Comment réagir quand une branche s'éternise :**

- **Découper la branche** en plusieurs branches plus petites, chacune avec un périmètre clairement délimité
- **Rebase régulièrement** sur `main` pour ne pas accumuler les conflits :

```bash
git fetch origin
git rebase origin/main
```

- **Signaler à l'équipe** si la branche est bloquée en attente de review ou d'une décision externe

**Exception :** les branches `experiment/` ne sont pas soumises à cette règle — elles peuvent durer le temps de l'exploration et n'ont pas vocation à être mergées telles quelles.

---

## Merge Requests

Quand le travail sur une branche est terminé, on ouvre une **Merge Request** (GitLab) ou **Pull Request** (GitHub / Azure DevOps) pour faire relire et intégrer ses modifications dans `main`.

**Avant d'ouvrir une Merge Request :**

1. Vérifier que les tests passent : `uv run pytest`
2. Vérifier que le linting passe : `uv run ruff check .`
3. Vérifier que les docstrings sont présentes : `uv run interrogate .`
4. S'assurer que le pre-commit passe : `uv run pre-commit run --all-files`

**Bonnes pratiques :**

- Désigner les **bons reviewers** — au moins une personne qui connaît le code touché
- S'appuyer sur le pre-commit et le CI/CD pour garantir la qualité du code avant la revue
- Écrire une description claire de ce que fait la Merge Request et pourquoi
- **Squash les commits au merge** si la branche contient des commits intermédiaires type `wip` ou `fix typo` — l'historique de `main` doit rester lisible. Sur GitLab et GitHub, cette option est disponible directement dans l'interface au moment du merge.

---

## Issues

Les **Issues** (GitLab) ou **Issues** (GitHub) servent à tracer les demandes de changement, les bugs et les tâches à réaliser. Elles sont le point d'entrée pour toute modification du projet.

**Quand créer une issue :**

- On a trouvé un bug à corriger
- On veut demander une nouvelle fonctionnalité
- On souhaite signaler un problème à l'équipe responsable de la branche `main`

**Bonnes pratiques :**

- Donner un titre clair et descriptif
- Décrire le problème ou la demande avec suffisamment de contexte pour qu'un autre développeur puisse comprendre et agir
- Associer l'issue à une Merge Request quand le travail est en cours

---

## Vérifier la présence du Wiki et du CI/CD

En arrivant sur un projet, vérifier que le dépôt dispose d'un **wiki** et d'un **pipeline CI/CD**. Si l'un des deux est absent, il faut le signaler et le mettre en place.

### Wiki

- **GitHub / GitLab** : le wiki n'est pas activé par défaut. Si absent, aller dans **Settings** (ou **Paramètres**) du dépôt et activer l'option Wiki.
- **Azure DevOps** : un espace Wiki est disponible d'office dans chaque projet, accessible depuis le menu latéral. Rien à activer.

**Si le wiki existe déjà**, il faut le maintenir à jour tout au long du projet. Un wiki est un dépôt Git à part entière : pour le modifier, on le clone comme un projet normal, on édite les pages en local, puis on push ses modifications.

```bash
# Cloner le wiki (l'URL se termine par .wiki.git)
git clone git@github.com:<orga>/<projet>.wiki.git        # GitHub
git clone git@gitlab.com:<orga>/<projet>.wiki.git        # GitLab

# Éditer les fichiers Markdown, puis pousser
cd <projet>.wiki
git add .
git commit -m "update wiki"
git push
```

Sur **Azure DevOps**, le wiki peut être édité directement depuis l'interface web ou cloné de la même manière via l'URL fournie dans la section Wiki du projet.

### Pipeline CI/CD

Vérifier si un pipeline existe déjà :

- **GitHub** : chercher un dossier `.github/workflows/` à la racine du dépôt.
- **GitLab** : chercher un fichier `.gitlab-ci.yml` à la racine.
- **Azure DevOps** : vérifier la section **Pipelines** dans le menu du projet et chercher un fichier `.azure-pipelines.yml` à la racine.

Si aucun pipeline n'est en place, il faut en créer un. Un projet sans CI/CD accumule de la dette technique invisible.

**Exemple concret : déploiement automatique du wiki.** Sur certains projets, le pipeline CI/CD est configuré pour prendre le contenu du dossier `docs/` et le pousser automatiquement vers le wiki du projet à chaque push sur `main`. Cela garantit que la documentation reste à jour sans intervention manuelle. Pour plus de détails, consulter la documentation officielle de votre plateforme : [GitHub Actions](https://docs.github.com/en/actions), [GitLab CI/CD](https://docs.gitlab.com/ee/ci/).

---

## Le projet a un `pyproject.toml`

C'est le **cas idéal**. Le projet est déjà configuré avec les standards modernes. Toutes les dépendances et leurs versions sont déclarées proprement.

```bash
# 1. Créer et activer l'environnement virtuel
uv venv
source .venv/Scripts/activate         # Git Bash
# ou : .venv\Scripts\activate         # CMD
# ou : .venv\Scripts\Activate.ps1     # PowerShell

# 2. Installer les dépendances
uv sync
```

`uv sync` installe toutes les dépendances aux versions exactes définies dans `uv.lock` et garantit que l'environnement est identique à celui des autres développeurs du projet.

Si le projet utilise pre-commit, installer également les hooks :

```bash
uv run pre-commit install
```

C'est tout. L'environnement est prêt.

---

## Le projet a un `requirements.txt`

C'est le cas le plus courant sur les projets plus anciens ou qui n'ont pas encore migré vers `uv`. Le fichier `requirements.txt` liste les dépendances du projet, mais sa qualité varie énormément d'un projet à l'autre.

### Premier réflexe : examiner le contenu du `requirements.txt`

Ouvrir le fichier et observer comment les dépendances sont déclarées. Il y a deux situations très différentes :

### Situation A — Versions spécifiées

```
pandas==2.1.4
scikit-learn==1.3.2
numpy==1.26.2
requests==2.31.0
```

Chaque dépendance a une version exacte (avec `==`). C'est correct : on sait précisément quelles versions ont été utilisées. La migration vers `uv` est simple et sans risque.

**Commandes :**

```bash
# 1. Initialiser le pyproject.toml
uv init

# 2. Créer et activer l'environnement virtuel
uv venv
source .venv/Scripts/activate         # Git Bash
# ou : .venv\Scripts\activate         # CMD
# ou : .venv\Scripts\Activate.ps1     # PowerShell

# 3. Importer les dépendances avec leurs versions exactes
uv add $(cat requirements.txt)

# 4. Vérifier l'installation
uv sync
uv run python -c "import pandas; print(pandas.__version__)"

# 5. Ajouter les outils de développement
uv add --dev ruff pre-commit pytest interrogate
uv run pre-commit install
```

Comme les versions sont épinglées, `uv` installera exactement les mêmes versions que celles déclarées dans le `requirements.txt`. Le risque de casse est quasi nul.

---

### Situation B — Versions absentes ou vagues

C'est la situation la plus courante, surtout sur les projets anciens ou qui n'ont jamais fait l'objet d'une mise au propre des dépendances. Beaucoup de projets ont été initialisés avec un simple `pip install pandas` sans jamais figer les versions.

```
pandas
scikit-learn
numpy
requests
```

ou :

```
pandas>=2.0
scikit-learn>=1.3
numpy
requests
```

Dans ce cas, les versions ne sont pas verrouillées. Cela signifie que deux développeurs qui installent le projet à des dates différentes obtiendront potentiellement des versions différentes de chaque bibliothèque. C'est une source fréquente de bugs difficiles à diagnostiquer : « Ça marche chez moi mais pas chez toi. »

**C'est le premier problème à régler.** Avant même de commencer à coder, il faut reconstruire un environnement reproductible. L'objectif : passer d'un `requirements.txt` vague à un `pyproject.toml` + `uv.lock` qui verrouille tout.

**Commandes :**

```bash
# 1. Initialiser le pyproject.toml
uv init

# 2. Créer et activer l'environnement virtuel
uv venv
source .venv/Scripts/activate         # Git Bash
# ou : .venv\Scripts\activate         # CMD
# ou : .venv\Scripts\Activate.ps1     # PowerShell

# 3. Importer les dépendances (uv résoudra les dernières versions compatibles)
uv add $(cat requirements.txt)

# 4. Vérifier l'installation
uv sync

# 5. IMPORTANT : tester que le projet tourne avec les versions résolues
uv run python -m pytest        # si des tests existent
uv run python src/main.py      # ou lancer le script principal

# 6. En cas d'incompatibilité, contraindre les versions manuellement
# Exemple : si le code ne fonctionne pas avec pandas 2.2
uv add "pandas>=2.0,<2.2"

# 7. Une fois que tout fonctionne, ajouter les outils de développement
uv add --dev ruff pre-commit pytest interrogate
uv run pre-commit install
```

> **Attention :** comme les versions d'origine ne sont pas épinglées, `uv` installera les **dernières versions compatibles** au moment de la migration. Il est impératif de **tester l'ensemble du projet** avant de valider. Si un script plante ou si des tests échouent, c'est probablement une incompatibilité de version — il faut alors contraindre la version dans le `pyproject.toml` avec `uv add "package>=x.y,<x.z"`.

---

### Dans les deux situations : finaliser la migration

Une fois que tout fonctionne, committer les nouveaux fichiers :

```bash
git add pyproject.toml uv.lock .python-version
git commit -m "migrate dependencies from requirements.txt to pyproject.toml"
```

> **Faut-il supprimer le `requirements.txt` ?** Pas toujours. Avant de le supprimer, vérifier si un autre fichier du projet en dépend. Par exemple, un `Dockerfile` contient souvent une ligne `RUN pip install -r requirements.txt`, et certains services cloud (Google Cloud Functions, AWS Lambda) s'attendent à trouver un `requirements.txt` pour installer les dépendances au déploiement. Dans ces cas, il faut **garder le `requirements.txt`** en parallèle du `pyproject.toml`. On peut le regénérer automatiquement à partir du `uv.lock` avec :
>
> ```bash
> uv export --format requirements-txt > requirements.txt
> ```
>
> Si rien ne référence le `requirements.txt`, on peut le supprimer sans risque.

---

## Le projet n'a pas de requirements

Certains projets anciens n'ont ni `pyproject.toml`, ni `requirements.txt`. Les dépendances ne sont documentées nulle part — il faut les déduire du code.

C'est la situation la plus délicate. Voici comment procéder :

**Étape 1 — Initialiser le projet et créer l'environnement :**

```bash
uv init
uv python pin 3.12                    # fixer la version Python
uv venv
source .venv/Scripts/activate         # Git Bash
# ou : .venv\Scripts\activate         # CMD
# ou : .venv\Scripts\Activate.ps1     # PowerShell
```

**Étape 2 — Identifier les dépendances en lisant le code :**

On utilise la commande `grep` pour chercher toutes les lignes d'import dans les fichiers Python. Voici comment elle fonctionne :

**Imports d'un seul fichier :**

```bash
grep -h "^import\|^from" mon_fichier.py
```

**Imports de tous les fichiers Python d'un dossier** (par exemple `src/`) :

```bash
grep -rh "^import\|^from" src/ --include="*.py" | sort -u
```

**Autre exemple** — les imports dans un dossier `fichier-exemple/` :

```bash
grep -rh "^import\|^from" fichier-exemple/ --include="*.py" | sort -u
```

**Explication de chaque option :**

| Option | Rôle |
|---|---|
| `-r` | Recherche récursive dans tous les sous-dossiers |
| `-h` | Masque le nom du fichier dans la sortie (affiche seulement la ligne trouvée) |
| `"^import\|^from"` | Cherche les lignes qui commencent par `import` ou `from` (le `^` signifie "début de ligne") |
| `--include="*.py"` | Ne cherche que dans les fichiers `.py` |
| `sort -u` | Trie les résultats et supprime les doublons (`-u` = unique) |

**Exemple de sortie :**

```
from pathlib import Path
import json
import os
import pandas as pd
from sklearn.model_selection import train_test_split
```

Il faut ensuite distinguer :
- Les modules de la **bibliothèque standard** (comme `os`, `json`, `pathlib`) → rien à installer
- Les **bibliothèques tierces** (comme `pandas`, `sklearn`, `requests`) → à ajouter au projet

**Étape 3 — Ajouter les dépendances identifiées :**

```bash
uv add pandas scikit-learn numpy requests  # adapter selon ce qui a été trouvé
```

**Étape 4 — Tester et ajuster :**

Exécuter le code et corriger les erreurs d'import manquants. Répéter jusqu'à ce que le projet tourne sans erreur.

> **Conseil :** ce travail de reconstruction des dépendances est le moment idéal pour mettre en place proprement les outils qualité (ruff, pre-commit, pytest). On repart sur des bases saines.
