# Tests avec pytest

Les tests automatisés garantissent que le code fait ce qu'il est censé faire — et continue de le faire après chaque modification. Ce chapitre couvre pytest, l'outil standard pour tester du code Python.

---

## Sommaire

- [Installation](#installation)
- [Structure des tests](#structure-des-tests)
- [Écrire un test](#écrire-un-test)
- [Fixtures](#fixtures)
- [Parametrize](#parametrize)
- [Mocking](#mocking)
- [Coverage](#coverage)
- [Configuration](#configuration)
- [Commandes utiles](#commandes-utiles)
- [Bonnes pratiques](#bonnes-pratiques)

---

## Installation

```bash
uv add --dev pytest pytest-cov
```

---

## Structure des tests

Les tests vivent dans le dossier `tests/` à la racine du projet, en miroir de la structure `src/` :

```
mon-projet/
├── src/
│   └── mon_projet/
│       ├── preprocessing.py
│       └── model.py
└── tests/
    ├── __init__.py
    ├── test_preprocessing.py
    └── test_model.py
```

Les fichiers de test commencent par `test_`, les fonctions de test aussi. pytest les découvre automatiquement.

---

## Écrire un test

```python
# src/mon_projet/preprocessing.py
def normalize(values: list[float]) -> list[float]:
    min_val = min(values)
    max_val = max(values)
    return [(v - min_val) / (max_val - min_val) for v in values]
```

```python
# tests/test_preprocessing.py
from mon_projet.preprocessing import normalize

def test_normalize_basic():
    result = normalize([0, 5, 10])
    assert result == [0.0, 0.5, 1.0]

def test_normalize_already_normalized():
    result = normalize([0.0, 1.0])
    assert result == [0.0, 1.0]

def test_normalize_raises_on_empty():
    with pytest.raises(ValueError):
        normalize([])
```

### Assertions utiles

```python
# Égalité simple
assert result == expected

# Valeurs flottantes (éviter == sur des floats)
assert result == pytest.approx(0.333, rel=1e-3)

# Exceptions
with pytest.raises(ValueError, match="liste vide"):
    normalize([])

# Type
assert isinstance(result, list)

# Contenu d'une collection
assert 42 in result
assert len(result) == 3
```

---

## Fixtures

Une fixture est une fonction qui prépare un contexte réutilisable entre plusieurs tests : données d'entrée, connexion à une base, fichier temporaire…

```python
# tests/test_preprocessing.py
import pytest
from mon_projet.preprocessing import normalize

@pytest.fixture
def sample_values():
    return [10.0, 20.0, 30.0, 40.0, 50.0]

def test_normalize_length(sample_values):
    result = normalize(sample_values)
    assert len(result) == len(sample_values)

def test_normalize_bounds(sample_values):
    result = normalize(sample_values)
    assert min(result) == 0.0
    assert max(result) == 1.0
```

### Fixtures dans conftest.py

Pour partager des fixtures entre plusieurs fichiers de tests, les placer dans `tests/conftest.py` — pytest le charge automatiquement :

```python
# tests/conftest.py
import pytest
import pandas as pd

@pytest.fixture
def sample_dataframe():
    return pd.DataFrame({
        "age": [25, 30, 35, None],
        "salary": [50000, 60000, 70000, 80000],
    })

@pytest.fixture
def tmp_csv(tmp_path):
    """Fixture utilisant tmp_path (fixture built-in de pytest)."""
    csv_file = tmp_path / "data.csv"
    csv_file.write_text("a,b\n1,2\n3,4\n")
    return csv_file
```

### Scopes de fixture

Par défaut, une fixture est recréée pour chaque test. Le scope permet de la partager :

```python
@pytest.fixture(scope="module")   # une fois par fichier de test
def db_connection():
    conn = create_connection()
    yield conn
    conn.close()

@pytest.fixture(scope="session")  # une fois pour toute la session de tests
def heavy_model():
    return load_model("weights.pkl")
```

---

## Parametrize

`@pytest.mark.parametrize` permet de lancer le même test avec plusieurs jeux de données, sans dupliquer le code :

```python
import pytest
from mon_projet.preprocessing import normalize

@pytest.mark.parametrize("values,expected", [
    ([0, 10], [0.0, 1.0]),
    ([5, 5, 5, 10], [0.0, 0.0, 0.0, 1.0]),
    ([-10, 0, 10], [0.0, 0.5, 1.0]),
])
def test_normalize(values, expected):
    assert normalize(values) == pytest.approx(expected)
```

pytest génère un test distinct pour chaque jeu de données — les erreurs sont rapportées individuellement, ce qui facilite le débogage.

---

## Mocking

Le mocking remplace temporairement un composant réel (une API externe, une base de données, l'horloge système) par un faux contrôlable dans les tests.

```bash
uv add --dev pytest-mock
```

### Mocker une fonction

```python
# src/mon_projet/data_loader.py
import requests

def fetch_data(url: str) -> dict:
    response = requests.get(url)
    response.raise_for_status()
    return response.json()
```

```python
# tests/test_data_loader.py
from mon_projet.data_loader import fetch_data

def test_fetch_data(mocker):
    # Remplace requests.get par un faux qui retourne une réponse contrôlée
    mock_response = mocker.Mock()
    mock_response.json.return_value = {"key": "value"}
    mock_response.raise_for_status.return_value = None
    mocker.patch("mon_projet.data_loader.requests.get", return_value=mock_response)

    result = fetch_data("https://example.com/api")
    assert result == {"key": "value"}
```

### Mocker une exception

```python
def test_fetch_data_handles_error(mocker):
    mocker.patch(
        "mon_projet.data_loader.requests.get",
        side_effect=requests.exceptions.ConnectionError
    )
    with pytest.raises(requests.exceptions.ConnectionError):
        fetch_data("https://example.com/api")
```

### Mocker le temps

```python
def test_timestamp(mocker):
    mocker.patch("mon_projet.utils.datetime").now.return_value = datetime(2024, 1, 1)
    result = get_current_timestamp()
    assert result == "2024-01-01"
```

---

## Coverage

La coverage mesure le pourcentage de lignes de code exécutées par les tests. Elle indique les zones non testées — pas nécessairement un problème, mais un signal utile.

```bash
# Lancer les tests avec coverage
uv run pytest --cov=src --cov-report=term-missing

# Générer un rapport HTML (plus lisible)
uv run pytest --cov=src --cov-report=html
# Ouvrir htmlcov/index.html dans le navigateur
```

Sortie typique :

```
Name                              Stmts   Miss  Cover   Missing
---------------------------------------------------------------
src/mon_projet/preprocessing.py     12      2    83%   45-46
src/mon_projet/model.py             34      8    76%   78-85
---------------------------------------------------------------
TOTAL                               46     10    78%
```

### Fixer un seuil minimal

```bash
# Faire échouer la CI si la coverage tombe sous 80%
uv run pytest --cov=src --cov-fail-under=80
```

---

## Configuration

Centraliser la configuration pytest dans `pyproject.toml` pour ne pas avoir à répéter les options à chaque commande :

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = [
    "--tb=short",           # tracebacks courts
    "--cov=src",            # coverage activée par défaut
    "--cov-report=term-missing",
]

[tool.coverage.run]
source = ["src"]
omit = [
    "*/tests/*",
    "*/__init__.py",
]

[tool.coverage.report]
fail_under = 80
show_missing = true
```

---

## Commandes utiles

```bash
# Lancer tous les tests
uv run pytest

# Lancer un fichier spécifique
uv run pytest tests/test_preprocessing.py

# Lancer un test spécifique
uv run pytest tests/test_preprocessing.py::test_normalize_basic

# Lancer les tests avec un mot-clé dans le nom
uv run pytest -k "normalize"

# Afficher les tests qui auraient été lancés (sans les lancer)
uv run pytest --collect-only

# Arrêter au premier échec
uv run pytest -x

# Afficher les print() dans les tests
uv run pytest -s

# Mode verbeux
uv run pytest -v
```

---

## Bonnes pratiques

**Un test = une chose à vérifier.** Un test qui vérifie dix comportements à la fois est difficile à déboguer. Si un test échoue, on doit savoir exactement pourquoi sans lire vingt lignes d'assertions.

**Nommer les tests de manière explicite.** `test_normalize_returns_zero_for_min_value` est infiniment plus utile que `test_normalize_2` quand un test échoue en CI à 2h du matin.

**Tester les cas limites en priorité.** Les valeurs nulles, les listes vides, les chaînes vides, les nombres négatifs, les doublons — c'est là que les bugs se cachent, pas dans le cas nominal.

**Ne pas tester le code des bibliothèques tierces.** Tester que `pandas.read_csv` lit bien un CSV n'a aucun intérêt. Tester que votre fonction qui appelle `read_csv` gère correctement un fichier malformé, oui.

**Garder les tests rapides.** Les tests lents (intégration, appels réseau, chargement de modèles) doivent être séparés et marqués :

```python
@pytest.mark.slow
def test_full_pipeline():
    ...
```

```bash
# Lancer uniquement les tests rapides
uv run pytest -m "not slow"
```
