# Contributing

Merci de l'intérêt que vous portez à ce projet. Toute contribution est la bienvenue, qu'il s'agisse de corriger une coquille, de mettre à jour une commande obsolète ou d'ajouter un nouveau chapitre.

---

## Ce que vous pouvez contribuer

- **Corrections** : fautes, commandes incorrectes, liens morts, captures d'écran obsolètes
- **Mises à jour** : nouvelles versions d'outils, changements de comportement, meilleures pratiques émergentes
- **Ajouts** : nouveaux chapitres, exemples supplémentaires, cas d'usage manquants
- **Traductions** : adaptation du contenu à d'autres langues

---

## Comment contribuer

### 1. Signaler un problème

Si vous repérez une erreur ou une lacune sans vouloir la corriger vous-même, ouvrez une [issue](../../issues) en décrivant :

- où se situe le problème (fichier et section)
- ce qui est incorrect ou manquant
- ce que vous attendriez à la place

### 2. Proposer une modification

```bash
# Forker le dépôt puis cloner votre fork
git clone https://github.com/Mastocodeur/learn-basics-repo.git
cd learn-basics-repo

# Créer une branche dédiée
git checkout -b fix/nom-du-correctif
# ou
git checkout -b feat/nom-de-la-fonctionnalite

# Faire vos modifications, puis
git add .
git commit -m "fix: correction de la commande uv sync dans 05a"
git push origin fix/nom-du-correctif
```

Ouvrez ensuite une **Pull Request** vers la branche `main` en décrivant brièvement ce que vous avez changé et pourquoi.

---

## Conventions rédactionnelles

Pour garder le wiki cohérent, merci de respecter les conventions suivantes.

**Langue** : le wiki est rédigé en français. Les termes techniques courants (commit, branch, hook…) restent en anglais.

**Format** : chaque fichier est en Markdown. Les titres suivent une hiérarchie stricte (`#` pour le titre principal, `##` pour les sections, `###` pour les sous-sections). Pas de titre de niveau 4 ou plus.

**Blocs de code** : toujours spécifier le langage après les trois backticks (` ```bash `, ` ```python `, ` ```toml `…). Les commandes destinées au terminal sont en `bash`.

**Ton** : direct et concis. On s'adresse au lecteur avec « vous » (pas « tu »). Pas de formules de politesse superflues.

**Liens internes** : utiliser des chemins relatifs (`[05a — La base : uv](05a_UV.md)`), pas des URLs absolues.

---

## Ce que nous n'acceptons pas

- Contenu lié à un outil propriétaire ou à un contexte d'entreprise spécifique
- Modifications qui réduisent la généralité du wiki (exemples trop ciblés, configurations non portables)
- Ajouts sans explication du cas d'usage

---

## Questions

Pour toute question avant de contribuer, ouvrez une issue avec le label `question`. Nous répondrons dans les meilleurs délais.
