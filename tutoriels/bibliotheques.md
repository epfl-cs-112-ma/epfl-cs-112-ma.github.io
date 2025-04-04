---
title: Bibliothèques
layout: default
use_mathjax: true
---

# Bibliothèques

Dans ce chapitre, nous voyons plus en détail comment exploiter les bibliothèques externes disponibles sur Internet.
Nous verrons notamment la notion de *package manager*.
De plus, nous apprendrons à nous servir un peu plus de `uv` pour gérer la liste des bibliothèques utilisées par un projet.

> Note de terminologie : en anglais, le mot utilisé est *library*.
> Sa traduction en français est "bibliothèque".
> Cependant, par erreur de faux amis, énormément de francophones parlent de "librairies" pour faire référence aux bibliothèques informatiques.
> Ne vous étonnez donc pas si vous rencontrez ce terme en français.
> La traduction anglaise correcte de "librairie", à savoir *bookstore*, n'est en revanche jamais utilisée pour désigner des bibliothèques informatiques.

**Ce chapitre ne fait pas partie de la "matière d'examen".**
Vous aurez besoin de ce chapitre pour votre projet.
En revanche, comme son exploitation requiert l'usage de recherche Internet, il ne sera pas évalué en situation d'examen.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Rappels

Jusqu'à présent, vous avez sans doute utilisé la commande `uv add` "aveuglément", sans réellement savoir ce qu'elle fait.
Nous vous avons montré deux façons de l'utiliser.
D'une part, en mode `--dev` pour installer mypy et pytest :

```
$ uv add --dev mypy pytest
```

D'autre part, en mode "normal" pour installer Arcade :

```
$ uv add arcade
```

En milieu de projet, nous vous avons recommandé de "mettre à jour" Arcade avec la commande suivante :

```
$ uv add "arcade>=3.0.2"
```

Vous avez peut-être "vu" une partie de leurs conséquences dans votre fichier `pyproject.toml`.
Peut-être `git diff` vous a alerté sur les changements que `uv add` opère dans ce fichier.
Normalement, le fichier `pyproject.toml` de votre projet doit ressembler à ceci :

```toml
[project]
name = "platformer"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "arcade>=3.0.2",
]

[tool.mypy]
strict = true

[dependency-groups]
dev = [
    "mypy>=1.15.0",
    "pytest>=8.3.4",
]
```

On peut y voir deux sections qui semblent avoir un lien avec les commandes `uv add`.
Pour les dépendances normales, nous avons

```toml
dependencies = [
    "arcade>=3.0.2",
]
```

qui dit que votre projet "a besoin", d'une manière ou d'une autre, de la bibliothèque nommée `arcade`, dans une version au moins égale à `3.0.2`.

Mais *d'où vient* cette bibliothèque ?
Comment `uv` l'a-t-elle trouvée, et comment l'a-t-elle mise à disposition de votre projet (de sorte que vous puissiez faire `import arcade`) ?

## Python Package Index (PyPI)

Le [Python Package Index (PyPI)](https://pypi.org/) est un service sur Internet qui recense toutes les bibliothèques disponibles pour Python.
La plupart des langages modernes ont un service similaire.

Cherchons "Arcade" dans la barre de recherche de PyPI pour tomber [sur la page PyPI de `arcade`](https://pypi.org/project/arcade/).
On y trouve de nombreuses informations utiles, notamment :

* la dernière version en date (tiens, c'est 3.1.0, on va pouvoir la mettre à jour) ;
* un lien vers la documentation (que vous connaissez déjà, j'espère) ;
* un lien vers le repository GitHub public où est développée la bibliothèque ;
* une "commande rapide" pour l'installer : `pip install arcade`.

À propos de ce dernier point, rappelez-vous que dans [l'introduction au projet](/projet/01-decouverte.html#installation-de-arcade), nous avons attiré votre attention sur le fait que la plupart des bibliothèques vous disent d'utiliser `pip` pour les installer.
Or, nous vous faisons utiliser `uv add` à la place.
Nous allons encore revenir sur ce point un peu plus bas.

Non seulement PyPI propose aux humains de chercher des bibliothèques qui les intéressent, mais il permet aussi aux *machines* de trouver les différentes versions des bibliothèques existantes.
Il possède une API (Application Programming Interface) qui permet aux outils comme `uv` de se renseigner sur les bibliothèques existantes, leurs versions, et les URLs d'où elles peuvent être téléchargées pour être installées.

## Dépendances

Rappelez-vous qu'en installant Arcade, uv avait installé une série d'autres bibliothèques : `pyglet`, `pymunk`, etc.
Ces bibliothèques étaient nécessaires car `arcade` en *dépend*.
Chaque bibliothèque déclare sa liste de dépendances : d'autres bibliothèques (et leurs numéros de version minimale associée) dont elles ont besoin pour fonctionner.

Quelque part dans le code d'Arcade, il y a un `import pyglet`.
Si on n'installait que `arcade`, mais pas `pyglet`, on aurait des problèmes d'imports manquants.
C'est pourquoi `arcade` déclare une dépendance sur `pyglet`.
Lorsque uv installe `arcade`, elle voit cette dépendance et installe donc `pyglet` aussi, ainsi que, potentiellement, toutes ses dépendances transitives.
Vous n'avez donc pas à vous en soucier.

## Ce que fait vraiment `uv add`

Lorsque vous avez fait `uv add arcade` dans votre projet, voici ce qui s'est réellement produit :

1. `uv` demande à PyPI quelle est la version la plus récente de la bibliothèque nommée `arcade` ;
2. PyPI répond (au début du projet) "3.0.1" ;
3. `uv` demande à PyPI à quelle URL il peut télécharger `arcade` version 3.0.1 ;
4. PyPI répond avec l'URL https://files.pythonhosted.org/packages/02/21/e5fdb07584f31b66b139af4722ae2a040cb5510b32c9d067d14bbeab7dd6/arcade-3.0.1.tar.gz#sha256=dd3869b1d1ca5e8cf2a50fd38f6caeac83fa88290f2692a349e6b4277247cac3
5. `uv` télécharge et extrait son contenu dans un dossier séparé sur votre ordinateur (pas directement dans votre projet, mais à un endroit unique) ;
6. `uv` inspecte le contenu du fichier spécial `arcade.egg-info/requires.txt` pour y trouver les dépendances de `arcade` ;
7. Il trouve (entre autres), `pyglet~=2.1.0`, et répète donc le processus avec cette bibliothèque ;
8. Lorsqu'il a épuise toutes les dépendances transitives, `uv` fait une copie (enfin presque) du contenu de toutes les bibliothèques dont votre projet a besoin dans le dossier `.venv/` de votre projet.
9. `uv` stocke aussi dans le fichier `uv.lock` les versions exactes des bibliothèques qu'il a ainsi "résolues", de sorte que `uv sync` installe *exactement* les mêmes bibliothèques plus tard chez votre binôme (ou toute autre personne qui voudrait travailler sur le projet) ;
9. `uv` est maintenat prêt, ainsi que VS Code, a vous donner accès à ces bibliothèques.

Pour éviter des téléchargements inutiles, uv garde une copie de chaque bibliothèque qu'il télécharge.
Ces copies sont stockées une fois pour toutes sur votre ordinateur, avec le numéro de version correspondant.
Si plusieurs projets utilisent la même version de la même bibliothèque, uv ne doit donc pas la retélécharger.

En revanche, uv met à disposition *uniquement les bibliothèques et versions requises* par un projet dans le `.venv/` de ce projet.
De cette façon, si vous travaillez sur plusieurs projets sur le même ordinateur, chaque projet est *isolé* des bibliothèques des autres projets.
C'est particulièrement important quand deux projets dépendent de versions différentes, *incompatibles*, de la même bibliothèque !

La "mise à disposition" pour chaque projet ressemble à une *copie* dans le dossier `.venv/`, mais en fait uv crée des liens ("hard links", si ça vous intéresse), de sorte à ne pas consommer plusieurs fois de la place sur votre disque dur.

uv nous donne le meilleur des deux mondes :

* on ne garde qu'une seule copie de chaque bibliothèque par ordinateur ;
* mais les projets sont quand même bien isolés les uns des autres.

## Un mot sur `pip install` et les virtual envs

Au contraire d'`uv`, `pip install` installe *globalement* les bibliothèques que vous lui demandez.
Cela cause de gros soucis, parmi lesquels :

* pas de trace des bibliothèques réellement requises par un projet ;
* conflits entre versions incompatibles de bibliothèques utilisées par plusieurs projets sur un ordinateur.

Pour éviter ces problèmes, un usage *responsable* de `pip` nécessite *absolument* de travailler à chaque fois dans ce que Python appelle un "virtual env" (environnement virtuel).
L'installation "globale" se transforme alors en installation à l'intérieur du virtual env.
C'est infiniment mieux que les problèmes ci-dessus, mais cela vient quand même avec le problème de la duplication des téléchargements et de l'espace disque nécessaire.

C'est pourquoi `uv` est une bien meilleure alternative, de plus en plus utilisée parmi les développeurs et développeuses Python.
Non seulement il protège des problèmes ci-dessus, mais en plus il évite la duplication de données sur votre disque dur.

Il ne faut jamais utiliser directement `pip` dans un projet géré par `uv`.
À chaque fois qu'un site (ou votre IA favorite) vous dit d'utiliser `pip install`, vous devez utiliser `uv add` à la place.

## Dépendances de développement

Pour Arcade, nous avons utilisé `uv add`.
Pour pytest et mypy, nous avons plutôt utilisé `uv add -dev`.
Cette variante ajoute les bibliothèques au groupe de développement seulement.

Le groupe de développement contient les outils qui sont nécessaires pendant le *développement* de votre projet, mais pas pendant son exécution.
Vous, vous avez besoin de `mypy` et `pytest` pour travailler sur votre projet.
Mais si vous deviez le distribuer à vos connaissances, qui veulent seulement jouer à votre jeu, elles n'en ont pas besoin.
Elles ont cependant besoin d'`arcade`, et des autres dépendances transitives d'`arcade`.

## Autres commandes d'`uv`

```bash
$ uv sync
```

Réajuste votre environnement pour correspondre à votre `pyproject.toml` et `uv.lock`.

`uv` exécute implicitement cette commande à chaque `uv add` et `uv run`.
Néanmoins, si vous passez toujours par VS Code pour lancer votre projet et ses tests, vous devrez au moins lancer `uv sync` après chaque mise à jour de `pyproject.toml`/`uv.lock`, y compris si vous avez reçu ces changements de votre bionôme *via* un `git pull`.

**Rappel :** `uv` est la source de vérité en ce qui concerne [la validité de votre projet](/projet/05-refactoring-1.html#uv-est-la-source-de-vérité).

```bash
$ uv add "lib>=1.2.3"
```

Installe `lib`, au moins à la version 1.2.3.

Ceci est la manière standard de *mettre à jour* une bibliothèque que vous aviez déjà installée.
Par défaut, `uv add lib` gardera la version enregistrée dans `uv.lock`.

```bash
$ uv remove lib
```

Désinstalle `lib` du projet.

À utiliser si vous n'avez plus besoin d'une biliothèque.

```bash
$ uv remove --dev lib
```

Désinstalle `lib` du groupe des dépendances de développement.
