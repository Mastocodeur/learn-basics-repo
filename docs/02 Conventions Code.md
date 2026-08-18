# Conventions de code Python

Passons en revue les conventions de code Python. 

Ce guide présente les conventions de nommage, de style et de formatage à respecter dans les projets Python, en s'appuyant sur le standard [PEP 8](https://peps.python.org/pep-0008/).

Les outils [Ruff](https://docs.astral.sh/ruff/) et [pre-commit](https://pre-commit.com/) permettent de vérifier et d'appliquer automatiquement la majorité de ces règles, cf : [Configuration d'un projet](<05 Routine Projet.md>)

---

## Sommaire

- [Les PEP : définition](#les-pep--définition)
- [PEP 8 : le style de code Python](#pep-8--le-style-de-code-python)
- [Conventions de nommage](#conventions-de-nommage)
- [Indentation et espaces](#indentation-et-espaces)
- [Imports](#imports)
- [Commentaires et docstrings](#commentaires-et-docstrings)
- [Longueur de ligne](#longueur-de-ligne)
- [Bonnes pratiques générales](#bonnes-pratiques-générales)
  - [Type hints](#type-hints--annotations-de-type-pep-484)
  - [Comparaisons](#comparaisons)
  - [F-strings](#f-strings-pep-498)
  - [Gestion des chemins](#gestion-des-chemins)
- [Conventions au-delà de Python](#conventions-au-delà-de-python)
- [Récapitulatif rapide](#récapitulatif-rapide)

---

## Les PEP : définition

Les **PEP** (*Python Enhancement Proposals*) sont des documents de référence qui décrivent les évolutions du langage Python. Chaque PEP a un numéro et un rôle précis. Ils sont rédigés par la communauté Python et approuvés par les mainteneurs du langage. On peut les considérer comme les « règles du jeu » auxquelles tout développeur Python se conforme.

Voici les PEP les plus importants à connaître pour un projet data science :

| PEP | Sujet |
|-----|-------|
| [PEP 8](https://peps.python.org/pep-0008/) | Style de code Python (le plus connu) |
| [PEP 20](https://peps.python.org/pep-0020/) | Le Zen de Python (`import this`) |
| [PEP 257](https://peps.python.org/pep-0257/) | Conventions de docstrings |
| [PEP 484](https://peps.python.org/pep-0484/) | Type hints |
| [PEP 621](https://peps.python.org/pep-0621/) | Métadonnées de projet dans `pyproject.toml` |

---

## PEP 8 : le style de code Python

Le PEP 8 est le guide de style officiel de Python, publié en 2001 et maintenu depuis. Il définit les règles de formatage, de nommage et d'organisation du code. Son objectif principal : **la lisibilité**. L'idée est qu'un développeur qui découvre un projet pour la première fois puisse le lire sans effort, car le code suit les mêmes conventions partout.

> *"Le code est lu beaucoup plus souvent qu'il n'est écrit."* — Guido van Rossum

Concrètement, le PEP 8 couvre tout ce qui est détaillé dans les sections suivantes : comment nommer ses variables, comment indenter, comment organiser ses imports, etc. Plutôt que de tout retenir par cœur, l'approche recommandée est de configurer des outils comme **Ruff** qui vérifient et corrigent automatiquement le code (cf. [Configuration d'un projet](05 Routine Projet.md)).

---

## Conventions de nommage

Le nommage est probablement l'aspect le plus visible des conventions de code. Un nom bien choisi rend le code auto-documenté : on comprend ce que fait une variable ou une fonction rien qu'en la lisant. En Python, plusieurs styles de "case" coexistent, chacun réservé à un type d'élément précis.

### Les différents styles

Le tableau ci-dessous présente les principaux styles de casse utilisés en programmation. Chaque style a un usage conventionnel qu'il est important de respecter pour que le code reste cohérent et lisible par tous les développeurs.

| Style | Nom | Exemple | Usage principal |
|-------|-----|---------|-----------------|
| `snake_case` | Snake case | `ma_variable`, `calculer_total` | Variables, fonctions, modules Python |
| `PascalCase` | Pascal case | `MonModele`, `DataProcessor` | Classes Python |
| `UPPER_SNAKE_CASE` | Constante | `MAX_RETRY`, `API_URL` | Constantes (API = *Application Programming Interface*, interface de programmation ; URL = *Uniform Resource Locator*, adresse web) |
| `camelCase` | Camel case | `maVariable`, `getData` | JavaScript, Java (non utilisé en Python) |
| `kebab-case` | Kebab case | `mon-projet`, `data-loader` | Noms de fichiers non-Python, URLs, CSS |

### Application en Python

Le tableau suivant détaille la convention à appliquer pour chaque type d'élément en Python. La colonne « Exemple incorrect » montre les erreurs les plus fréquentes, souvent dues à des habitudes prises dans d'autres langages (Java, JavaScript).

| Élément | Convention | Exemple correct | Exemple incorrect |
|---------|-----------|-----------------|-------------------|
| Variable | `snake_case` | `user_name` | `userName`, `UserName` |
| Fonction | `snake_case` | `load_data()` | `LoadData()`, `loadData()` |
| Classe | `PascalCase` | `DataProcessor` | `data_processor`, `dataProcessor` |
| Méthode | `snake_case` | `def get_result(self):` | `def GetResult(self):` |
| Constante | `UPPER_SNAKE_CASE` | `MAX_ITERATIONS = 100` | `max_iterations`, `MaxIterations` |
| Module (fichier .py) | `snake_case` | `data_loader.py` | `DataLoader.py`, `data-loader.py` |
| Package (dossier) | `snake_case` | `mon_projet/` | `MonProjet/`, `mon-projet/` |
| Variable privée | `_snake_case` | `_internal_cache` | `internalCache` |
| Variable "très privée" | `__snake_case` | `__secret_key` | — |
| Variable jetable | `_` | `for _ in range(10):` | `for i in range(10):` (si `i` non utilisé) |

### Noms à éviter

Au-delà du style de casse, certains noms sont à proscrire car ils introduisent de la confusion ou des bugs silencieux :

- Les lettres seules `l`, `O`, `I` comme noms de variable (confusion avec `1`, `0`)
- Les noms trop courts sans contexte : `x`, `d`, `tmp` (sauf dans des boucles courtes ou des formules mathématiques)
- Les noms trop longs : `the_total_number_of_items_in_the_list`
- Les noms qui masquent des *built-ins* (fonctions natives de Python, disponibles sans import) : `list`, `dict`, `type`, `input`, `id`, `sum`, `min`, `max`

Le dernier point est particulièrement piégeux. Si on nomme une variable `list`, on écrase la fonction native `list()` de Python, ce qui peut provoquer des erreurs difficiles à diagnostiquer plus loin dans le code :

```python
# Incorrect
list = [1, 2, 3]           # masque le built-in list()
type = "admin"             # masque le built-in type()
l = 10                     # confusion avec 1

# Correct
items = [1, 2, 3]
user_type = "admin"
length = 10
```

---

## Indentation et espaces

L'indentation et l'usage des espaces sont des éléments fondamentaux en Python. Contrairement à d'autres langages où l'indentation est purement esthétique, en Python elle fait partie de la syntaxe : une mauvaise indentation provoque une erreur à l'exécution.

### Indentation

Python utilise **4 espaces** par niveau d'indentation. Ne jamais utiliser de tabulations. Cette règle est universelle dans l'écosystème Python et les éditeurs modernes (VS Code, Cursor) sont configurés par défaut pour insérer 4 espaces lorsqu'on appuie sur la touche Tab.

```python
# Correct
def calculate_total(prices):
    total = 0
    for price in prices:
        if price > 0:
            total += price
    return total

# Incorrect (2 espaces)
def calculate_total(prices):
  total = 0
  for price in prices:
    if price > 0:
      total += price
  return total
```

### Espaces autour des opérateurs

Les opérateurs d'affectation (`=`), de comparaison (`==`, `>`, `<`), et arithmétiques (`+`, `-`, `*`) doivent être entourés d'un espace de chaque côté. Cela améliore considérablement la lisibilité, surtout dans les expressions complexes :

```python
# Correct
x = 5
y = x + 3
result = x * 2 + y
is_valid = x > 0 and y < 10

# Incorrect
x=5
y = x+3
result = x * 2+y
```

### Espaces dans les appels de fonction

Dans les appels de fonction, les parenthèses ne doivent pas contenir d'espace superflu. Les virgules séparant les arguments sont suivies d'un espace, mais pas précédées. Pour les arguments nommés (`kwarg=value`), il n'y a pas d'espace autour du `=` :

```python
# Correct
print(x)
my_list = [1, 2, 3]
my_dict = {"key": "value"}
func(arg1, arg2, kwarg="value")

# Incorrect
print( x )
my_list = [1,2,3]
my_dict = {"key" : "value"}
func(arg1 , arg2 , kwarg = "value")
```

### Lignes vides

Les lignes vides servent à séparer visuellement les blocs logiques du code. Le PEP 8 définit des règles précises pour leur usage :

- **2 lignes vides** avant et après une définition de fonction ou de classe au niveau du module
- **1 ligne vide** entre les méthodes d'une classe

L'exemple suivant illustre ces deux règles :

```python
import os


def first_function():
    pass


def second_function():
    pass


class MyClass:

    def method_one(self):
        pass

    def method_two(self):
        pass
```

---

## Imports

La gestion des imports est un point souvent négligé, mais qui a un impact direct sur la lisibilité d'un fichier Python. Un fichier dont les imports sont désordonnés ou inutiles est plus difficile à comprendre et à maintenir.

### Ordre des imports

Le PEP 8 impose un ordre précis pour les imports. Ils sont organisés en **3 groupes**, séparés par une ligne vide. Cet ordre permet de distinguer immédiatement d'où vient chaque dépendance :

1. Bibliothèque standard (modules intégrés à Python)
2. Bibliothèques tierces (installées via pip/uv)
3. Imports locaux (modules du projet)

```python
# 1. Bibliothèque standard
import os
from pathlib import Path

# 2. Bibliothèques tierces
import pandas as pd
from sklearn.model_selection import train_test_split

# 3. Imports locaux
from src.data.loader import load_dataset
from src.utils import clean_text
```

### Règles

En plus de l'ordre, quelques règles supplémentaires encadrent la façon d'écrire les imports. La plus importante : chaque instruction `import` ne doit importer qu'un seul module. En revanche, il est acceptable d'importer plusieurs éléments depuis un même module avec `from ... import` :

```python
# Correct — un import par ligne
import os
import sys

# Incorrect — plusieurs modules sur une ligne
import os, sys

# Correct — plusieurs éléments d'un même module
from pathlib import Path, PurePath

# À éviter — import wildcard (importe tout et pollue le namespace,
# c'est-à-dire l'espace de noms du module courant)
from os import *
```

Ruff trie et organise automatiquement les imports avec la règle `I`, qui intègre le comportement d'**isort** (un outil dédié au tri des imports).

---

## Commentaires et docstrings

Les commentaires et les docstrings sont deux mécanismes distincts pour documenter le code. Les commentaires (précédés de `#`) s'adressent aux développeurs qui lisent le code source. Les docstrings (entre `"""`) s'adressent aux utilisateurs du code, c'est-à-dire ceux qui appellent les fonctions et classes sans forcément lire leur implémentation.

### Commentaires

Un bon commentaire explique le **pourquoi**, pas le **quoi**. Le code lui-même doit être suffisamment lisible pour que son fonctionnement soit évident. Un commentaire qui reformule le code est redondant et finit par devenir trompeur quand le code évolue mais pas le commentaire.

Voici la différence entre un commentaire utile et un commentaire inutile :

```python
# Correct — explique le pourquoi
# Taux de TVA au 01/01/2024, source : legifrance.gouv.fr
tax_rate = 0.20

# Incorrect — décrit le quoi (redondant avec le code)
# On met 0.20 dans tax_rate
tax_rate = 0.20
```

### Docstrings (PEP 257)

Les docstrings documentent les modules, classes et fonctions. Elles sont placées immédiatement après la définition, entre triple guillemets (`"""`). Contrairement aux commentaires, les docstrings sont accessibles à l'exécution via l'attribut `__doc__` et sont utilisées par les outils de génération de documentation automatique.

Il existe plusieurs formats de docstrings (Google, NumPy, reStructuredText). Le format **Google** est recommandé pour sa lisibilité.

**Fonction simple** — une seule ligne suffit lorsque la fonction est triviale :

```python
def add(a: int, b: int) -> int:
    """Retourne la somme de deux entiers."""
    return a + b
```

**Fonction complexe (format Google)** — dès qu'une fonction accepte des paramètres non triviaux, il est recommandé de détailler chaque argument, la valeur de retour et les exceptions éventuelles :

```python
def load_data(filepath: str, encoding: str = "utf-8") -> pd.DataFrame:
    """Charge un fichier CSV dans un DataFrame.

    Args:
        filepath: Chemin vers le fichier CSV.
        encoding: Encodage du fichier. Par défaut "utf-8".

    Returns:
        DataFrame contenant les données du fichier.

    Raises:
        FileNotFoundError: Si le fichier n'existe pas.
    """
    return pd.read_csv(filepath, encoding=encoding)
```

**Classe** — la docstring de la classe décrit son rôle et ses attributs. Chaque méthode possède ensuite sa propre docstring :

```python
class DataProcessor:
    """Classe de traitement et normalisation des données.

    Attributes:
        data: Liste de valeurs numériques à traiter.
    """

    def __init__(self, data: list[float]):
        """Initialise le processeur avec les données fournies.

        Args:
            data: Liste de valeurs numériques.
        """
        self.data = data
```

L'extension VS Code **autoDocstring** permet de générer automatiquement le squelette d'une docstring au format Google : il suffit de taper `"""` sous la signature d'une fonction et l'extension pré-remplit les sections `Args`, `Returns` et `Raises` à partir des paramètres et type hints.

L'outil **interrogate** permet de vérifier que toutes les fonctions et classes possèdent une docstring (cf. [Configuration d'un projet](05 Routine Projet.md)).

---

## Longueur de ligne

Le PEP 8 recommande une limite de **79 caractères** par ligne, héritée de l'époque des terminaux 80 colonnes. En pratique, avec les écrans modernes, la plupart des projets utilisent une limite de **120 caractères**, ce qui offre un bon compromis entre lisibilité et espace disponible. Cette valeur est configurée dans `pyproject.toml` via `[tool.ruff] line-length = 120`.

Lorsqu'une ligne dépasse cette limite, il faut la découper. Python offre plusieurs mécanismes pour cela.

### Techniques de découpage

**Parenthèses implicites** — c'est la méthode recommandée. Python considère que tout ce qui est entre parenthèses, crochets ou accolades peut s'étendre sur plusieurs lignes :

```python
# Correct
result = (
    first_value
    + second_value
    - third_value
)

filtered = [
    item
    for item in collection
    if item.is_valid()
]
```

**Appels de fonction longs** — le même principe s'applique naturellement aux fonctions avec beaucoup de paramètres. La virgule après le dernier argument (*trailing comma*, ou « virgule finale ») est recommandée car elle facilite les futurs ajouts sans modifier la ligne précédente :

```python
# Correct
df = pd.read_csv(
    filepath,
    sep=";",
    encoding="utf-8",
    parse_dates=["date_column"],
)
```

**Backslash** — le caractère `\` permet de continuer une ligne, mais cette méthode est fragile (un espace invisible après le `\` casse la syntaxe) et moins lisible. À n'utiliser que quand les parenthèses implicites ne sont pas possibles :

```python
# Acceptable, mais les parenthèses implicites sont préférées
total = first_value \
    + second_value \
    + third_value
```

---

## Bonnes pratiques générales

Cette section regroupe plusieurs pratiques idiomatiques de Python qui, sans être strictement liées au PEP 8, contribuent à écrire un code plus propre et plus maintenable.

### Type hints — annotations de type (PEP 484)

Les annotations de type (appelées *type hints* en anglais) permettent d'indiquer le type attendu des arguments et de la valeur de retour d'une fonction. Elles n'ont aucun effet à l'exécution (Python reste un langage dynamique), mais elles améliorent considérablement la lisibilité et permettent aux outils d'analyse statique (comme `mypy` ou `pyright`) de détecter des erreurs avant même d'exécuter le code.

#### Types de base

```python
def greet(name: str) -> str:
    return f"Bonjour, {name}"

def calculate_mean(values: list[float]) -> float:
    return sum(values) / len(values)

def is_valid(age: int) -> bool:
    return age >= 0
```

#### Types optionnels et unions

```python
# Valeur pouvant être None (deux syntaxes équivalentes)
def find_user(user_id: int) -> dict | None:   # Python 3.10+
    ...

from typing import Optional
def find_user(user_id: int) -> Optional[dict]: # Python < 3.10
    ...

# Union de types
def process(value: int | str) -> str:
    return str(value)
```

#### Collections

```python
# Listes, ensembles, tuples
def tag_items(items: list[str]) -> set[str]: ...
def get_coordinates() -> tuple[float, float]: ...

# Dictionnaires
def count_words(text: str) -> dict[str, int]: ...

# Générateurs et itérables
from collections.abc import Iterator, Iterable, Generator

def read_lines(path: str) -> Iterator[str]: ...
def process_all(items: Iterable[int]) -> list[int]: ...
```

#### TypedDict — typer les dictionnaires structurés

Quand un dictionnaire a une structure fixe et connue, `TypedDict` permet de la décrire précisément :

```python
from typing import TypedDict

class UserRecord(TypedDict):
    id: int
    name: str
    email: str
    active: bool

def get_user(user_id: int) -> UserRecord:
    return {"id": user_id, "name": "Alice", "email": "alice@example.com", "active": True}
```

#### Dataclasses — alternative orientée objet aux TypedDict

Pour des structures de données plus riches, les dataclasses combinent le typage et la commodité :

```python
from dataclasses import dataclass, field

@dataclass
class ModelConfig:
    learning_rate: float = 0.001
    n_epochs: int = 100
    batch_size: int = 32
    features: list[str] = field(default_factory=list)

config = ModelConfig(learning_rate=0.01, features=["age", "salary"])
print(config.learning_rate)  # 0.01
```

#### Protocol — typer par comportement (duck typing)

`Protocol` permet de typer par interface plutôt que par héritage — si un objet a les bonnes méthodes, il est compatible :

```python
from typing import Protocol

class Saveable(Protocol):
    def save(self, path: str) -> None: ...

def export(obj: Saveable, path: str) -> None:
    obj.save(path)

# Toute classe avec une méthode save(path) est acceptée,
# sans avoir à hériter de Saveable
```

#### Callable — typer les fonctions passées en argument

```python
from collections.abc import Callable

def apply(func: Callable[[float], float], values: list[float]) -> list[float]:
    return [func(v) for v in values]

# func doit être une fonction qui prend un float et retourne un float
apply(lambda x: x ** 2, [1.0, 2.0, 3.0])
```

#### Vérification statique avec mypy

`mypy` analyse le code sans l'exécuter et signale les incohérences de types :

```bash
uv add --dev mypy

# Vérifier le projet
uv run mypy src/
```

```python
def add(a: int, b: int) -> int:
    return a + b

add(1, "deux")  # mypy signale : Argument 2 to "add" has incompatible type "str"; expected "int"
```

Configuration dans `pyproject.toml` :

```toml
[tool.mypy]
python_version = "3.12"
strict = false          # true pour activer toutes les vérifications
ignore_missing_imports = true
```

> **Conseil** : ne pas chercher à tout typer dès le début. Commencer par les signatures de fonctions publiques — c'est là que le typage apporte le plus de valeur à un coût raisonnable.

### Comparaisons

Python propose des idiomes spécifiques pour les comparaisons qui sont à la fois plus lisibles et plus performants que leurs alternatives. Par exemple, pour vérifier si une variable est `None`, on utilise `is` (test d'identité) plutôt que `==` (test d'égalité), car `None` est un singleton en Python (il n'existe qu'une seule instance de `None` en mémoire). De même, pour tester si une liste est vide, on exploite directement la valeur booléenne de la liste plutôt que de vérifier sa longueur :

```python
# Correct
if x is None:
if x is not None:
if not items:          # liste vide
if items:              # liste non vide
if isinstance(x, int):

# Incorrect
if x == None:
if x != None:
if len(items) == 0:
if len(items) > 0:
if type(x) == int:
```

### F-strings (PEP 498)

Les f-strings, introduites en Python 3.6, sont la méthode recommandée pour formater des chaînes de caractères. Elles sont plus lisibles, plus concises et plus performantes que les anciennes approches (concaténation avec `+` ou méthode `.format()`). On les reconnaît au préfixe `f` devant la chaîne :

```python
name = "Alice"
age = 30

# Correct — f-string
message = f"{name} a {age} ans"

# À éviter — concaténation
message = name + " a " + str(age) + " ans"

# À éviter — .format()
message = "{} a {} ans".format(name, age)
```

### Gestion des chemins

Le module `pathlib`, disponible depuis Python 3.4, offre une API orientée objet pour manipuler les chemins de fichiers. Il est préférable au module `os.path` car il produit un code plus lisible et gère automatiquement les différences entre systèmes d'exploitation (séparateurs `/` vs `\`). L'opérateur `/` remplace avantageusement `os.path.join()` :

```python
from pathlib import Path

# Correct
data_path = Path("data") / "raw" / "dataset.csv"
if data_path.exists():
    content = data_path.read_text()

# À éviter
import os
data_path = os.path.join("data", "raw", "dataset.csv")
if os.path.exists(data_path):
    with open(data_path) as f:
        content = f.read()
```

---

## Conventions au-delà de Python

Les conventions de nommage ne sont pas propres à Python. Chaque langage, outil ou technologie possède ses propres usages. Dans un projet data science, on est souvent amené à écrire du SQL, du YAML, du JSON, ou à nommer des branches Git. Le tableau suivant récapitule les conventions les plus courantes pour chacun de ces contextes. Les respecter permet de rester cohérent avec les pratiques de chaque écosystème :

| Contexte | Convention | Exemple |
|----------|-----------|---------|
| SQL (*Structured Query Language* — langage de requêtes pour bases de données) : tables, colonnes | `snake_case` | `user_id`, `created_at` |
| SQL : mots-clés | `MAJUSCULES` | `SELECT`, `FROM`, `WHERE` |
| YAML (*YAML Ain't Markup Language* — format de fichiers de configuration) : clés | `kebab-case` ou `snake_case` | `api-key`, `max_retry` |
| JSON (*JavaScript Object Notation* — format d'échange de données) : clés | `camelCase` ou `snake_case` | `userName`, `user_name` |
| Variables d'environnement | `UPPER_SNAKE_CASE` | `DATABASE_URL`, `API_KEY` |
| Fichiers Markdown | `UPPER_SNAKE_CASE` ou `kebab-case` | `README.md`, `guide-installation.md` |
| Branches Git | `kebab-case` | `feature/add-login`, `fix/data-loader` |
| Commits Git | `<type>: <description>`, impératif, minuscules | `feat: add user authentication` |
| CSS (*Cascading Style Sheets* — feuilles de style pour pages web) : classes | `kebab-case` | `main-container`, `btn-primary` |
| JavaScript | `camelCase` (variables/fonctions), `PascalCase` (classes/composants) | `getUserData()`, `UserCard` |

---

## Récapitulatif rapide

Ce tableau synthétique peut servir de référence rapide à garder sous la main lors du développement :

```
Variables, fonctions, modules  →  snake_case
Classes                        →  PascalCase
Constantes                     →  UPPER_SNAKE_CASE
Fichiers Python                →  snake_case.py
Packages Python                →  snake_case/
Branches Git                   →  kebab-case
Variables d'environnement      →  UPPER_SNAKE_CASE
SQL tables/colonnes            →  snake_case
SQL mots-clés                  →  MAJUSCULES

Indentation                    →  4 espaces
Longueur de ligne              →  120 caractères
Imports                        →  standard → tiers → locaux
Docstrings                     →  format Google
Type hints                     →  toujours si possible
```
