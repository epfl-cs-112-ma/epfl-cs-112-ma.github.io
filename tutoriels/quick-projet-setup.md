---
title: Setup rapide d'un projet
layout: default
---

# Setup rapide d'un projet

Cette petite référence rappelle les instructions de création d'un nouveau projet.
Vous en aurez besoin à chaque séance d'exercices, ainsi que pour votre gros projet.

## Création du projet en tant que tel

```
$ pwd
.../projets
$ mkdir nouveau-projet
$ cd nouveau-projet
$ uv init -p 3.13
```

Ajouter à la fin de `pyprojet.toml` :

```
[tool.mypy]
strict = true
```

Intallez mypy et pytest :

```
$ uv add --dev mypy pytest
```

Créer le dossier `tests` et son fichier `__init__.py` :

```
$ mkdir tests
$ touch tests/__init__.py
```

Ajouter `-> None` dans la définition de `def main` dans `main.py` :

```diff
-def main():
+def main() -> None:
     print("Hello from solutions-serie-02!")
```

Vérifiez que mypy et pytest s'exécutent sans erreur :

```
$ uv run mypy .
$ uv run pytest
```

## Configuration git

(Optionnel)

Faire un commit avec l'état initial, avant de commencez votre travail :

```
$ git add .
$ git commit -m "Initial commit."
```

Créer un nouveau repo *privé* sur GitHub.

⚠️ Ne *pas* cocher la case "Add a readme file", ne *pas* sélectionner de "`.gitignore` template" ni de licence à ce stade. ⚠️

Une fois créé, récupérer l'URL SSH `git@...` du repo.
Connecter le nouveau projet à ce repo et pousser le commit initial.

```
$ git remote add origin git@...
$ git push -u origin main
```
