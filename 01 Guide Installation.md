# Guide d'installation 

Ce fichier a pour but de guider la configuration de ton pc et de t'aider à installer les différents outils recommandés.


## Sommaire

- [Installation Python](#installation-python)
- [Installation de VS Code et Cursor](#installation-de-vs-code-et-cursor)
- [Installation de git](#installation-de-git)
- [Installation UV](#installation-uv)
- [Installation de MSYS2](#installation-de-msys2)
- [Installation de Node JS](#installation-de-node-js)
- [Configuration de Git et SSH](#configuration-de-git-et-ssh)
- [Installation Divers](#installation-divers)
- [Redémarrer votre PC](#redemarrer-votre-pc)

**IMPORTANT** : Suivre rigoureusement les étapes dans l'ordre !


___
## Installation Python

Python : https://www.python.org/downloads/

(ajoutez python au PATH)

___
## Installation de VS Code et Cursor

VS Code : https://code.visualstudio.com/

Cursor : https://cursor.com/

Installer les extensions VS Code :
* Ruff, 
* autoDocstring, 
* Python, 
* Jupyter, 
* Copilot,
* PDF Viewer
* PPTX Preview
* Live Server
* Excel Viewer
* Tout autre extension qui vous semble utile...etc

## Installation de git

Installer git : https://git-scm.com/install/windows

Les points d'attention durant la configuration : 

    Select Components:
    * Windows Explorer integration
    * Git Bash Here
    * Git GUI Here (optionnel)
    * Git LFS (si proposé, recommandé)

    Default Editor
    * Choisir Visual Studio Code si disponible (Use Visual Studio Code as Gitt's default editor)

    Adjusting your PATH:
    * Git from the command line and also from 3rd-party software 
    ⚠️ Important pour que VS Code reconnaisse Git

    SSH executable : 
    * Use bundled OpenSSH 
    (le plus simple et compatible)

    HTTPS transport backend : 
    * Use the OpenSSL library 

    Line ending conversions : 
    * Checkout Windows-style, commit Unix-style line endings (recommended) 

    Terminal emulator : 
    * Use MinTTY (default terminal of MSYS2) 

    git pull behavior : 
    * Fast-forward or merge (default) 

    Credential helper :
    * Git Credential Manager 

    Extra options :
    * Enable file system caching
    * Enable symbolic links (optionnel)


Fermer complètement le CMD, et VS Code.

Puis vérification avec : 

```
git --version
```


___
## Installation UV

Guide sur ce lien : https://docs.astral.sh/uv/getting-started/installation/#__tabbed_1_2


Dans le Windows PowerShell : 

```
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
$env:Path = "C:\Users\<votre_username>\.local\bin;$env:Path"
# Exemple : $env:Path = "C:\Users\rgasmi\.local\bin;$env:Path"
```

Puis dans le menu : `Démarrer -> Modifier les variables d’environnement système -> Variables d’environnement`. (**Etape pouvant nécessiter les droits administrateur ou l’aide de l’équipe IT**)


Puis dans `Variables utilisateur`, sélectionner `Path -> Modifier` puis cliquez sur `Nouveau` et ajouter :

```
C:\Users\<votre_username>\.local\bin
# Exemple : C:\Users\rgasmi\.local\bin
```

On valide tout, on ferme tout, on relance et on teste la bonne mise en place avec : 

```
uv --version
```



___
## Installation de MSYS2

MSYS2 : https://www.msys2.org/

Suivre le guide

___
## Installation de Node JS

Node JS : https://nodejs.org/en/download

___
## Configuration de Git et SSH

Configurer ses identifiants Git :

```
git config --global user.name "Prénom Nom"
git config --global user.email "<votre_email>"
# Exemple : git config --global user.email "rgasmi@example.com"
```

Puis dans le CMD Windows, vérification de l'installation de SSH : 

```
ssh -V
```

Création de la clé SSH : 

```
ssh-keygen -t ed25519 -C "<votre_email>"
# Exemple : ssh-keygen -t ed25519 -C "rgasmi@example.com"
```

On appuie deux fois sur `enter`.

Cela a créé deux fichiers :

```
C:\Users\<votre_username>\.ssh\id_ed25519      ← clé privée (secrète)
C:\Users\<votre_username>\.ssh\id_ed25519.pub  ← clé publique (à partager)

# Exemple :
# C:\Users\rgasmi\.ssh\id_ed25519
# C:\Users\rgasmi\.ssh\id_ed25519.pub
```

On ajoute alors la clé publique dans la plateforme Git (GitHub, GitLab…) en la récupérant dans le CMD via la commande :

```
type %USERPROFILE%\.ssh\id_ed25519.pub
```

- **GitHub** : la coller dans `Settings → SSH and GPG keys → New SSH key`
- **GitLab** : la coller dans `Profile → Preferences → SSH Keys`

Vérification dans le CMD :

```
ssh -T git@github.com
# ou : ssh -T git@gitlab.com
```

On obtient normalement un message de bienvenue confirmant l'authentification.

On vérifie que la commande git soit acceptée et accessible dans VS Code :


On peut maintenant travailler sur nos repos depuis VS Code ! 

___
## Installation Divers

Installation de toute les autres applications qui peuvent être utiles : 

* Notion : https://www.notion.com/fr/desktop
* Postman : https://www.postman.com/downloads/
* OBS Studio (vivement conseillé pour vos futures démonstrations) : https://obsproject.com/fr/download
* Microsoft PowerToys (via Microsoft Store)
* etc.

___
## Redémarrer votre PC

Une fois que tout ça est réalisé, on redémarre le PC pour être sûr que tous les changements sont effectifs.

Ton pc est prêt ! 