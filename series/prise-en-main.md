---
title: Prise en main de Python et VS Code
layout: default
---

# Prise en main de Python et VS Code

Ce cours utilise le langage de programmation [Python](https://www.python.org/) pour illustrer les nouveaux concepts.
Dans cette première série, nous allons nous assurer de prendre en main ce nouveau langage et les outils associés.

Même si vous avez déjà utilisé Python auparavant, il est indispensable de suivre cette série.
Vous n'avez peut-être pas utilisé tous les outils que nous considérerons requis pour ce cours, et pour votre projet.

Nous utiliserons les outils suivants (il n'est pas nécessaire de suivre ces liens maintenant ; nous allons vous guider pas à pas) :

* [uv](https://docs.astral.sh/uv/) pour gérer nos projets et leurs dépendances (y compris certains des *autres* outils).
* [VS Code](https://code.visualstudio.com/) *(optionnel)* pour l'édition de nos programmes (remplaçant donc Geany par rapport au premier semestre).
* [mypy](https://mypy-lang.org/) *en mode strict* pour valider les types statiques de nos programmes.
* [pytest](https://docs.pytest.org/en/stable/) *(dans une série ultérieure)* pour *tester* nos programmes.

## Installation de `uv`

[uv](https://docs.astral.sh/uv/) sera notre outil à tout faire.
Il coordonnera l'installation de plusieurs autres outils, y compris Python lui-même !

### Sur les machines de l'EPFL

Si vous utilisez les machines de l'EPFL, ouvrez un *Terminal* et entrez la commande suivante.

*Rappel* : le `$` en début de ligne ne doit pas être copié !
Il indique que le reste de la ligne est une commande à exécuter dans le terminal.
Les lignes qui ne commencent pas par `$` sont des *résultats* attendus des commandes.

```bash
$ curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR=~/Desktop/myfiles/.local/bin sh
```

### Sur votre propre machine

Suivez les [instructions d'installation de uv](https://docs.astral.sh/uv/getting-started/installation/) pour votre système.

### Vérifiez votre installation

Dans les deux cas:

1. *Fermez le terminal* que vous avez utilisé pour installer uv.
2. Ouvrez un nouveau terminal, et exécutez :

```bash
$ uv version
uv 0.5.29 (ca73c4754 2025-02-05)
```

Vous devriez obtenir un résultat similaire à la deuxième ligne ci-dessus (celle sans `$`).

## Installation de VS Code et de ses extensions

Contrairement aux autres outils, nous n'imposons pas VS Code.
Si vous avez déjà l'habitude d'utiliser un environnement de développement intégré (IDE) pour Python, tel que [PyCharm](https://www.jetbrains.com/pycharm/), vous pouvez l'utiliser dans le cadre de ce cours.

En revanche, évitez les éditeurs de texte trop simples, tels que Geany, ne suffiront plus ce semestre (ni dans la suite de vos projets).
Les IDEs offrent de nombreuses fonctionnalités qui vont vous aider dans vos développements.

### Installer VS Code

#### Sur les machines de l'EPFL

Vous n'avez rien à faire ici.
VS Code est déjà installé.

#### Sur votre propre machine

Suivez les [instructions d'installation de VS Code](https://code.visualstudio.com/download) pour votre système.

### Installer les extensions

Ceci est à faire que vous utilisiez les machines de l'EPFL ou non.

1. Lancez VS Code.
2. Sur la gauche de l'écran, en haut, il y a une colonne d'icônes.
   Cliquez sur l'icône ![Icône Extension dans VS Code](/assets/img/icon-vs-code-extension.png).
   Le panneau suivant doit s'afficher :

   ![Panneau Extensions dans VS Code](/assets/img/vs-code-extension-pane.png)

3. Dans la barre de recherche, cherchez puis installez les extensions suivantes :

   * Python (éditeur : Microsoft)
     (cela installe automatiquement "Python Debugger" de Microsoft en plus)
   * Mypy Type Checker (éditeur : Microsoft)

   ![Extensions de VS Code pour Python](/assets/img/vs-code-python-extensions.png)

## Créer un projet Python

C'en est fini des installations.
Nous pouvons maintenant créer notre premier projet Python.

> Notez que nous parlons désormais de *projet*, et non plus de *programme*.
> Cela n'a rien à voir avec la notion de "projet" académique dans le cadre de vos cours.
> Dans l'informatique, chaque "programme" est généralement constitué de plusieurs morceaux.
> On appelle donc cela un *projet*.

### Un projet vierge

Dans un terminal, naviguez vers un dossier où vous rangerez tous vos projets pour le cours.
N'oubliez pas de le ranger dans `myfiles` si vous travaillez sur les machines de l'EPFL.
Nous ferons référence à ce dossier comme `projets/`.

Dans ce dossier, créez un projet Python vierge dans le sous-dossier `prise-en-main` avec les commandes suivantes :

```bash
projets$ mkdir prise-en-main
projets$ cd prise-en-main
prise-en-main$ uv init -p 3.13
Initialized project `prise-en-main`
```

L'argument `-p 3.13` à `uv init` signifie que nous voulons utiliser Python 3.13 dans ce projet.
S'il ne l'a pas encore fait, `uv` installera automatiquement un Python 3.13 à son usage.
C'est pour cela que nous n'avons pas dû intaller Python en tant que tel.
Même si votre système possède une version "globale" de Python, `uv` utilisera celle que nous avons exigée dans ce projet.
C'est très utile, car cela garantit que votre programme fonctionnera de la même manière sur n'importe quel ordinateur.

Vous pouvez d'ores et déjà lancer ce programme avec la commande suivante :

```bash
prise-en-main$ uv run hello.py
Using CPython 3.13.2
Creating virtual environment at: .venv
Hello from prise-en-main!
```

Les lignes "Using" et "Creating" ne s'afficheront que la première fois.
Si vous le lancez à nouveau, vous n'aurez plus que :

```bash
prise-en-main$ uv run hello.py
Hello from prise-en-main!
```

### Créer un commit git (optionnel)

`uv init` crée automatiquement un repository git pour le projet.
Si vous aviez besoin d'une preuve supplémentaire que tous les projets devraient utiliser git, la voici !

Vous pouvez créer un commit initial avec le contenu créé automatiquement par `uv init`.
Vous pourrez ainsi plus facilement voir ce que vous aurez modifié vous-mêmes dans des commits suivants.

Dans un terminal (rappel : sur Windows, dans "git bash"), utilisez les commandes :

```bash
$ git branch -m main              # nommer la branche principale 'main'
$ git add .                       # ajouter tous les fichiers à l'index
$ git commit -m "Initial commit." # créer le commit initial
```

Nous ne mentionnerons plus git à partir d'ici.
Charge à vous de l'utiliser à bon escient (même si ce n'est qu'en mode local, sans pousser sur GitHub), ou pas.

### Explorons le projet

Ouvrez maintenant VS Code (si ce n'est pas déjà fait), et utiliser le menu "File | Open Folder" (Ctrl+K Ctrl+O).
Sélectionnez le dossier `prise-en-main` et validez.
Vous devriez obtenir :

![Première ouverture de prise-en-main dans VS Code](/assets/img/vs-code-prise-en-main-premiere-ouverture.png)

Découvrons les fichiers que `uv init` a créés pour nous.

#### `hello.py`

C'est le fichier auquel on s'attend le plus.
Il s'agit du code source Python de notre projet :

```python
def main():
    print("Hello from prise-en-main!")


if __name__ == "__main__":
    main()
```

La ligne `def main():` introduit une fonction appelée `main`, sans argument.
Son implémentation contient l'appel d'une autre fonction, `print(...)`, qui affiche une valeur à l'écran.
Elle correspondrait à une ligne `cout << "..." << endl;` en C++.

Les deux dernière lignes disent ceci : si c'est ce fichier (`__name__`) qui est exécuté comme fichier principal du programme (`"__main__"`), alors appelle la fonction `main()`.
C'est ce qui se passe quand nous lançons `uv run hello.py`.
Contrairement à C++, la fonction `main()` n'est pas appelée automatiquement au démarrage du programme.
Ce sont ces deux dernières lignes qui s'en chargent explicitement.

Observons tout de suite une autre différence avec C++.
Les blocs de code ne sont pas entourés de `{}`.
À la place, ils sont introduits par un `:`, puis s'est *l'indentation* qui décide où se terminent les blocs !
Le `if` est donc bien en *dehors* de la `def main():`.

Si vous n'aviez pas encore pris le pli d'indenter correctement votre code, ça ne va plus tarder.
Python ne vous laissera tout simplement pas utiliser une mauvaise indentation !

#### `pyproject.toml`

Ce fichier contient la configuration globale de votre projet.
Pour l'instant, il ne contient que certaines méta-données qui n'ont que peu d'importance.
On note cependant une ligne qui déclare un pré-requis sur la version de Python nécessaire.

```toml
[project]
name = "prise-en-main"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = []
```

Plus tard, ce fichier déclarera des dépendances à des bibliothèques et outils externes.
Il contiendra aussi une configuration un peu plus avancée.

#### `README.md`

Un fichier [Markdown](https://docs.github.com/fr/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) contenant des informations à l'intention des utilisateurs et utilisatrices de votre projet.
Il n'entre pas en compte pour la compilation et l'exécution de votre programme.

Si vous poussez votre projet sur GitHub, le fichier `README.md` sera présenté comme "page d'accueil".

#### `.python-version`

Ce petit fichier enregistre de façon permanente la version de Python qui doit être utilisée par `uv`.
Vous ne devrez jamais le modifier.

#### `.gitignore`

Celui-ci est pour git.
Pour rappel, il indique à git certains fichiers qui ne doivent jamais être stockés par git.
Ces fichiers sont générés automatiquement par `uv`, par VS Code, ou par d'autres outils.

Vous n'aurez en principe pas à modifier ce fichier.

#### `uv.lock`

Ce fichier *ne doit jamais* être modifié à la main.
Il est entièrement manipulé par `uv` pour stocker les versions *exactes* des bibliothèques et outils utilisés par votre programme.

Il doit faire partie des fichiers stockés dans git, cependant.
En effet, cela permet de s'assurer que votre projet fonctionnera avec exactement les mêmes dépendances sur n'importe quel ordinateur.

### Lancer le programme depuis VS Code.

Il est parfois plus pratique de lancer votre projet directement depuis VS Code.
Lorsque le fichier `hello.py` est ouvert, cliquez sur l'icône ▷ située tout en haut à droite de l'écran.

## Activer le type checker mypy (obligatoire !)

Dans ce cours, nous utiliserons exclusivement du Python statiquement typé avec [mypy](https://mypy-lang.org/), *en mode strict*.
Configurons maintenant notre projet pour s'assurer que c'est le cas.

Ouvrez le fichier `pyproject.toml` dans VS Code.
Ajoutez les lignes suivantes à la fin :

```toml
[tool.mypy]
strict = true
```

et sauvegardez.

Puis, demandez à `uv` d'installer mypy pour votre projet avec la commande suivante :

```bash
$ uv add --dev mypy
Resolved 4 packages in 149ms
Installed 3 packages in 555ms
 + mypy==1.15.0
 + mypy-extensions==1.0.0
 + typing-extensions==4.12.2
```

Vous pouvez désormais lancer mypy sur votre projet avec :

```bash
$ uv run mypy . # le '.' est important
hello.py:1: error: Function is missing a return type annotation  [no-untyped-def]
hello.py:1: note: Use "-> None" if function does not return a value
hello.py:6: error: Call to untyped function "main" in typed context  [no-untyped-call]
Found 2 errors in 1 file (checked 1 source file)
```

Huh ! Malédiction !
Nous n'avons rien modifié dans le programme, mais il est déjà incorrect !

C'est normal.
Le programme Python généré par défaut par `uv init` ne contient pas de types statiques.
Il nous faut remédier à cela.

Dans VS Code, utiliser la combinaison de touche `Ctrl+Shift+P`, puis utilisez la barre de recherche pour trouver "Developer: Reload Window".
Après avoir fait des modifications dans votre `pyproject.toml`, il est parfois nécessaire d'utiliser cette commande pour que VS Code en tienne compte.

Ouvrez à nouveau `hello.py`.
Au bout d'un petit moment, si ce n'est pas immédiat, certaines lignes apparaîtront en rouge.
Ce sont les erreurs de mypy.

Corrigeons la déclaration de `def main` pour inclure le `-> None` exigé par mypy:

```diff
-def main():
+def main() -> None:
     print("Hello from prise-en-main!")
```

Après avoir sauvegardé, les lignes rouges disparaissent, indiquant que le problème est résolu.
On peut aussi s'en assurer en lançant mypy explicitement :

```bash
$ uv run mypy .
Success: no issues found in 1 source file
```

On voit ici un avantage certain de VS Code (ou de n'importe quel autre IDE) comparé à Geany.
Dès que l'on enregistre, VS Code adapte la présentation pour refléter les erreurs potentielles.
C'est un gain de temps incroyable lorsqu'on développe !
