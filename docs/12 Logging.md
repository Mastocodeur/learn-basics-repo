# Logging

Le logging permet de tracer ce que fait une application en cours d'exécution. C'est la différence entre un programme qui échoue silencieusement et un programme dont on comprend le comportement.

---

## Sommaire

- [Pourquoi pas print ?](#pourquoi-pas-print-)
- [Le module logging (bibliothèque standard)](#le-module-logging-bibliothèque-standard)
- [Loguru](#loguru)
- [Bonnes pratiques](#bonnes-pratiques)

---

## Pourquoi pas print ?

`print` convient pour du débogage rapide et local. Dès qu'une application tourne en production ou est partagée, il montre ses limites :

| | `print` | `logging` |
|---|---|---|
| Niveau de sévérité | ❌ aucun | ✅ DEBUG, INFO, WARNING, ERROR, CRITICAL |
| Horodatage | ❌ | ✅ |
| Nom du module source | ❌ | ✅ |
| Écriture dans un fichier | ❌ manuel | ✅ natif |
| Désactivable sans toucher au code | ❌ | ✅ |
| Filtrage par niveau | ❌ | ✅ |

---

## Le module logging (bibliothèque standard)

### Configuration minimale

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logger = logging.getLogger(__name__)

logger.debug("Message de débogage (non affiché au niveau INFO)")
logger.info("Démarrage du pipeline")
logger.warning("Fichier manquant, utilisation de la valeur par défaut")
logger.error("Erreur lors de la lecture du fichier")
logger.critical("Erreur fatale, arrêt du programme")
```

Sortie :

```
2024-03-15 10:23:45 | INFO     | mon_projet.pipeline | Démarrage du pipeline
2024-03-15 10:23:46 | WARNING  | mon_projet.pipeline | Fichier manquant, utilisation de la valeur par défaut
```

### Les niveaux de log

| Niveau | Valeur | Quand l'utiliser |
|---|---|---|
| `DEBUG` | 10 | Informations détaillées pour le débogage (valeurs intermédiaires, états…) |
| `INFO` | 20 | Confirmation que les choses se passent normalement (démarrage, étapes clés…) |
| `WARNING` | 30 | Quelque chose d'inattendu s'est produit, mais l'exécution continue |
| `ERROR` | 40 | Une erreur sérieuse — une fonctionnalité ne peut pas s'exécuter |
| `CRITICAL` | 50 | Erreur grave qui peut entraîner l'arrêt du programme |

### Un logger par module

La bonne pratique est de créer un logger par module avec `__name__`. Cela permet de savoir exactement d'où vient chaque message :

```python
# src/mon_projet/preprocessing.py
import logging

logger = logging.getLogger(__name__)
# __name__ vaut "mon_projet.preprocessing"

def clean_data(df):
    logger.info("Début du nettoyage : %d lignes", len(df))
    df = df.dropna()
    logger.debug("Après dropna : %d lignes", len(df))
    logger.info("Nettoyage terminé : %d lignes conservées", len(df))
    return df
```

### Écrire dans un fichier

```python
import logging

# Handler console
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)

# Handler fichier
file_handler = logging.FileHandler("app.log")
file_handler.setLevel(logging.DEBUG)

# Format commun
formatter = logging.Formatter(
    "%(asctime)s | %(levelname)s | %(name)s | %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)
console_handler.setFormatter(formatter)
file_handler.setFormatter(formatter)

# Logger racine
logging.basicConfig(level=logging.DEBUG, handlers=[console_handler, file_handler])
logger = logging.getLogger(__name__)
```

### Configurer le niveau depuis les variables d'environnement

```python
import logging
import os

log_level = os.environ.get("LOG_LEVEL", "INFO").upper()
logging.basicConfig(level=getattr(logging, log_level))
```

```bash
# En local : logs détaillés
LOG_LEVEL=DEBUG uv run python src/mon_projet/main.py

# En production : logs essentiels uniquement
LOG_LEVEL=WARNING uv run python src/mon_projet/main.py
```

---

## Loguru

[Loguru](https://github.com/Delgan/loguru) est une bibliothèque tierce qui simplifie drastiquement le logging Python. Elle remplace la configuration verbeuse du module standard par une API intuitive, avec des sorties colorées et des traces d'erreurs lisibles par défaut.

```bash
uv add loguru
```

### Utilisation de base

```python
from loguru import logger

logger.debug("Message de débogage")
logger.info("Démarrage du pipeline")
logger.warning("Valeur manquante détectée")
logger.error("Impossible de lire le fichier")
logger.critical("Erreur fatale")
```

Sortie colorée dans le terminal, avec horodatage, niveau et localisation automatiques — sans aucune configuration.

### Écrire dans un fichier avec rotation

```python
from loguru import logger

# Rotation automatique à 10 MB, conservation 7 jours, compression
logger.add(
    "logs/app.log",
    rotation="10 MB",
    retention="7 days",
    compression="zip",
    level="DEBUG",
    format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {name} | {message}",
)
```

### Capturer les exceptions avec le contexte complet

```python
from loguru import logger

@logger.catch
def process_data(df):
    """Le décorateur @logger.catch capture et logue toute exception avec sa trace."""
    return df["colonne_inexistante"].sum()

# Ou avec un bloc try/except
try:
    result = risky_operation()
except Exception:
    logger.exception("Erreur lors du traitement")  # logue avec la traceback complète
```

### Configurer le niveau depuis les variables d'environnement

```python
import os
from loguru import logger
import sys

log_level = os.environ.get("LOG_LEVEL", "INFO").upper()

# Supprimer le handler par défaut et en ajouter un configuré
logger.remove()
logger.add(sys.stderr, level=log_level)
```

### Loguru dans un projet avec plusieurs modules

Loguru utilise un logger global unique — pas besoin de `getLogger(__name__)`. Il suffit d'importer et d'utiliser :

```python
# Dans n'importe quel module
from loguru import logger

logger.info("Ce message inclut automatiquement le nom du fichier et la ligne")
```

### logging standard vs Loguru

| | `logging` | `loguru` |
|---|---|---|
| Configuration | Verbeuse | Minimale |
| Sortie colorée | ❌ (à configurer) | ✅ par défaut |
| Rotation de fichiers | Via `RotatingFileHandler` | `logger.add(..., rotation=)` |
| Capture d'exceptions | `logger.exception()` | `@logger.catch` ou `logger.exception()` |
| Compatibilité bibliothèques tierces | ✅ standard | À propager manuellement |
| Recommandé pour | Bibliothèques, projets avec intégrations | Scripts, pipelines, applications |

> **Note** : si vous développez une **bibliothèque** destinée à être importée par d'autres, utilisez le module `logging` standard et n'y ajoutez pas de handler — c'est à l'application finale de configurer le logging. Loguru est plus adapté aux **applications** et **pipelines**.

---

## Bonnes pratiques

**Ne jamais logger de données sensibles.** Les logs sont souvent écrits dans des fichiers, envoyés à des services de monitoring, ou stockés en clair. Une clé API ou un mot de passe dans les logs est une fuite de données.

```python
# À ne jamais faire
logger.info("Connexion avec le mot de passe : %s", password)

# À faire
logger.info("Connexion réussie pour l'utilisateur %s", username)
```

**Utiliser les niveaux correctement.** Un WARNING qui apparaît à chaque exécution normale n'est plus un warning — c'est du bruit qui masque les vrais problèmes. Réserver ERROR et CRITICAL pour les situations qui nécessitent une action.

**Logger les décisions, pas les étapes triviales.** `logger.debug("i = 42")` dans une boucle est inutile. `logger.info("Pipeline démarré avec %d fichiers", n)` est utile.

**Inclure le contexte dans les messages.** Un message comme `"Erreur"` n'aide personne. `"Erreur lors de la lecture de data/input.csv : fichier introuvable"` permet de diagnostiquer sans relancer.

```python
# Peu utile
logger.error("Erreur de lecture")

# Utile
logger.error("Impossible de lire '%s' : %s", filepath, str(e))
```

**Configurer le niveau via les variables d'environnement** (voir [chapitre 09 — Variables d'environnement](09_Secrets_Environnement.md)), pas en dur dans le code. Cela permet d'activer les logs DEBUG en production sans modifier le code ni redéployer.
