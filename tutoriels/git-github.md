---
title: Prise en main de git et GitHub
layout: default
---

# Prise en main de git et GitHub

[git](https://git-scm.com/) (prononcez "guitte" en translitération française) est ce qu'on appelle un système de contrôle de versions, ou Version Control System (VCS) en anglais.
Il permet de gérer les différentes versions d'un ensemble de fichiers.
On peut voir l'historique des changements, récupérer une version antérieure, etc.
De plus, il permet de *synchroniser* les historiques entre plusieurs ordinateurs.

Les VCS tels que git sont le plus connus pour permettre la *collaboration* entre plusieurs personnes sur un même projet.
Différentes personnes peuvent travailler chacune de leur côté sur une partie du projet.
Puis, git peut *fusionner* (merger) les changements faits de part et d'autre.

Nous verrons cette utilisation la semaine prochaine.
Nous vous encouragerons fortement à utiliser git pour collaborer avec votre binôme dans le cadre de votre projet.

Cependant, dans cette première approche, nous allons découvrir cet outil dans son utilisation la plus basique : garder des traces systématiques de l'évolution d'un projet pour soi-même.
Cela est bien sûr très utile pour vos projets informatiques, mais git peut aussi être utilisé efficacement avec n'importe quel type de fichier texte.
Par exemple, il est commun de l'utiliser avec des document LaTeX.
git vous sera donc utile dans l'ensemble de votre cursus.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Installation

Sur les machines de l'EPFL, git est déjà installé.

Sur votre propre machine, suivez les instructions correspondant à votre système [sur la page de download de git](https://git-scm.com/downloads).

Vous pouvez vérifier que git est correctement installé en exécutant la commande suivante dans un terminal.
Sous Windows, toutes les commandes git devront être exécutée dans le terminal "git bash" installé par git.
Les commandes ressembleront donc à un environnement Linux, peu importe votre système.

```bash
$ git -v
git version 2.40.1.windows.1
```

Le `$` au début de la ligne ne doit pas être recopié.
Il indique que le reste de la ligne est une commande à exécuter.
Les lignes qui ne commencent *pas* par un `$` sont la *sortie* attendue des commandes.

## Configuration globale

Avant toute chose, il faut configurer git avec votre nom et votre adresse e-mail.
Cette adresse e-mail sera *rendue publique* sur tout l'Internet.
Si vous avez une adresse sérieuse bien établie, elle peut être un bon choix.
Votre adresse `@epfl.ch` peut aussi servir.

Si vous êtes particulièrement contre révéler une adresse utile, créez une adresse que vous utiliserez pour toutes vos opérations git.

Exécutez les commandes suivantes dans un terminal :

```bash
$ git config --global user.name "Votre Nom"
$ git config --global user.email "votre.adresse@email.ch"
$ git config --global init.defaultBranch main
```

Les `"` font bien partie des commandes.

La troisième de ces commandes s'assure que votre "branche" par défaut sera nommée `main`, conformément aux conventions modernes.

### Windows

Sur Windows, exécutez également commande suivante :

```bash
$ git config --global core.autocrlf false
```

Par défaut, git essaye de vous "aider" à gérer les différences de fins de lignes entre Windows et les autres OS.
En effet, Windows utilise `\r\n` (les caractères dont les codes ASCII sont 13 et 10, respectivement) alors que Linux et MacOS n'utilisent que `\n`.
git pense qu'il est judicieux de convertir les fins de lignes en `\r\n` pour les utilisateurs Windows.
C'est malheureusement contre-productif.
Les éditeurs de code sur Windows sont parfaitement capables de gérer des `\n`, et le feront même par défaut.

### MacOS

Si vous avez déjà échangé des fichiers .zip entre MacOS et d'autres systèmes, vous savez sans doute que MacOS a la fâcheuse habitude de créer des fichiers `.DS_STORE` partout.
Ces fichiers ne devraient *jamais* être traités par git.
Nous allons donc configurer git pour qu'il les ignore systématiquement.

Créer un nouveau fichier texte appelé `global.gitignore` dans vos documents personnels, par exemple à `/Users/vous/global.gitignore`, avec l'unique ligne suivante :

```gitignore
.DS_STORE
```

Puis, dans un terminal, exécutez l'instruction suivante :

```bash
$ git config --global core.excludesfile /Users/vous/global.gitignore
```

en adaptant bien sûr le chemin.

## git en local

Nous allons d'abord explorer les commandes principales de `git` entièrement en local.
Il s'agira donc de tracer les évolutions d'un "projet", mais uniquement sur votre ordinateur, sans les synchroniser avec un autre ordinateur.

### Initialiser un repo

Commencez par créer un nouveau dossier vide `tuto-git`.
Dans ce dossier, *initialisez* un nouveau *repository* git (aussi connu sous les noms de "repo", "dépôt" ou "référentiel") avec `git init`.

```bash
$ pwd # où est-on exactement ?
/home/username
$ cd Desktop/myfiles/ # ou un autre dossier comme `Documents/`
$ mkdir projets-poo # crée un dossier projets-poo pour tous vos projets de ce cours
$ cd projets-poo # rentrer à l'intérieur de ce dossier
$ mkdir tuto-git
$ cd tuto-git
$ pwd
/home/username/Desktop/myfiles/projets-poo/tuto-git
$ git init
Initialized empty Git repository in .../tuto-git/.git/
```

⚠️ Vérifiez bien, par exemple avec `pwd`, que vous êtes dans le dossier attendu avant d'exécuter `git init`.

Le dossier `tuto-git`, et tous ses (futurs) sous-dossiers, est désormais considéré comme un repo git.
À tout moment, vous pouvez consulter l'état d'un repo avec `git status` :

```
$ git status
On branch main

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

On dirait qu'il ne se passe grand chose pour l'instant.
Changeons ça.

### Ajouter des fichiers

Créez un fichier texte `README.md` avec votre éditeur de texte préféré (cela peut être VS Code si vous l'avez déjà installé, ou Geany, etc.).
Vous pouvez commencer par y mettre le contenu suivant :

```md
# Tutoriel git

C'est mon premier repo git !
```

> L'extension `.md` correspond au format Markdown.
> C'est un format populaire pour écrire des documents texte qui peuvent être transformés en documents HTML simples.
> La ligne commençant par `#` est un titre principal (de niveau 1).

`git status` nous montre maintenant :

```
$ git status
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        README.md

nothing added to commit but untracked files present (use "git add" to track)
```

Notre nouveau fichier `README.md` est "untracked".
Cela veut dire que git ne connaît pas encore ce fichier.
La commande `git add` permet d'ajouter des fichiers à *l'index* de git :

```
$ git add README.md
$ git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   README.md
```

Le fichier est désormais "to be committed".
Il fait partie de l'index, mais pas encore d'un "commit".

### Créer un commit

L'index est une "zone de préparation".
On y prépare ce que l'on souhaite mettre dans un commit.
Un commit est une version pérenne, stable, d'un état de votre projet.

Créez maintenant votre premier commit avec

```
$ git commit -m "Initial commit."
[main (root-commit) 095ac8c] Initial commit.
 1 file changed, 3 insertions(+)
 create mode 100644 README.md
```

Le message après `-m` est un "commit message".
C'est une description des changements apportés à votre projet dans ce commit.
Si vous ne spécifiez pas cette option, git ouvrira un éditeur de texte pour vous donner l'occasion de l'écrire plus en détails.

```
$ git status
On branch main
nothing to commit, working tree clean
```

`git status` nous dit maintenant que notre "working tree" est "clean".
Il n'y a pas de changements par rapport au dernier commit.

### Les 3 états dans lesquels peuvent se trouver les fichiers

Il y a donc 3 états de votre projet dans les concepts de git :

* Le *working tree* est l'état de vos fichiers sur le système.
* L'*index* est la zone de préparation, où l'on rassemble les changements que l'on va pérenniser dans un commit.
* Le *head* est l'état de vos fichiers tels que stockés dans le dernier commit en date.

Lorsque vous faites `git status`, git vous montre *en vert* le contenu de l'index par rapport au head, et *en rouge* le working tree par rapport à l'index.

* `git add` intègre à l'index des changements du working tree ; il fait donc passer les changements du rouge au vert.
* `git commit` crée un nouveau commit avec le contenu de l'index ; il fait donc passer du vert à "rien du tout" (clean).

### Contenu d'un commit

Vous pouvez visualiser le dernier commit avec `git show` :

```
$ git show
commit 095ac8c1cdfabc61830ef35a034e70a598680f90 (HEAD -> main)
Author: Sébastien Doeraene <sjrdoeraene@gmail.com>
Date:   Sat Feb 15 17:30:00 2025 +0100

    Initial commit.

diff --git a/README.md b/README.md
new file mode 100644
index 0000000..e5feb10
--- /dev/null
+++ b/README.md
@@ -0,0 +1,3 @@
+# Tutoriel git
+
+C'est mon premier repo git !
```

Le commit contient les éléments suivants :

* Le *hash* (SHA-1) du commit : `095ac8c1cdfabc61830ef35a034e70a598680f90`.
  Il identifie de manière unique ce commit (en théorie, dans le monde entier !).
* L'*auteur* du commit, avec son adresse e-mail (oui, c'est donc mon adresse e-mail, publique, que vous voyez là ; vous la trouverez aussi traînant partout sur Internet).
* La *date* a laquelle ce commit a été créé.
* Le *commit message* qui décrit les changements apportés par ce commit.
* Le *diff* des changements apportés : les différences entre avant et après ce commit.

### Un deuxième commit

Apportons quelques changements dans notre fichier `README.md` et créons un deuxième commit.
Par exemple, ajoutez quelques lignes à la fin :

```md
# Tutoriel git

C'est mon premier repo git !

## Une section

Un paragraphe.
```

`git status` nous montre maintenant que `README.md` est *modifié* par rapport à l'index (et, *a fortiori*, par rapport au head) :

```
$ git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

Vous pouvez inspectez exactement en quoi le working tree diffère de l'index avec la commande `git diff` :

```diff
$ git diff
diff --git a/README.md b/README.md
index e5feb10..42b1051 100644
--- a/README.md
+++ b/README.md
@@ -1,3 +1,7 @@
 # Tutoriel git

 C'est mon premier repo git !
+
+## Une section
+
+Un paragraphe.
```

C'est très pratique lorsque vous développez !
Généralement, vous voudrez créer un nouveau commit à chaque fois que vous avez un état "stable" (typiquement : tous vos tests passent !).
Si vous cassez quelque chose, `git diff` peut alors vous montrer ce que vous avez modifié depuis cet état correct !

Créons un nouveau commit avec ces changements :

```
$ git add -u
$ git commit -m "Add one section."
[main 6feba6e] Add one section.
 1 file changed, 4 insertions(+)
```

`git add -u` veut dire "update".
Il intègre dans l'index toutes les *modifications* à des fichiers que git connaît déjà, mais sans ajouter les nouveaux fichiers.
Vous pouvez aussi utiliser `git add .` si vous voulez ajouter tout le working tree à l'index.

### `git log`

Nous arrivons finalement à l'intérêt majeur de git en mode "solo" : le *log*.
La commande `git log` vous montre un historique *complet* de tous les commits créés depuis le début de votre projet :

```
$ git log
commit 6feba6ed2064f43dcc3697fa3deea70b558094d1 (HEAD -> main)
Author: Sébastien Doeraene <sjrdoeraene@gmail.com>
Date:   Sat Feb 15 17:53:20 2025 +0100

    Add one section.

commit 095ac8c1cdfabc61830ef35a034e70a598680f90
Author: Sébastien Doeraene <sjrdoeraene@gmail.com>
Date:   Sat Feb 15 17:30:00 2025 +0100

    Initial commit.
```

Parfois, la version "one-line" est plus lisible :

```
$ git log --oneline
6feba6e (HEAD -> main) Add one section.
095ac8c Initial commit.
```

Vous pouvez inspecter le contenu complet de n'importe quel commit en spécifiant son hash (ou un préfixe de celui-ci ; souvent 5 à 7 caractères suffisent) :

```
$ git show 095ac8c
commit 095ac8c1cdfabc61830ef35a034e70a598680f90
Author: Sébastien Doeraene <sjrdoeraene@gmail.com>
Date:   Sat Feb 15 17:30:00 2025 +0100

    Initial commit.

diff --git a/README.md b/README.md
new file mode 100644
index 0000000..e5feb10
--- /dev/null
+++ b/README.md
@@ -0,0 +1,3 @@
+# Tutoriel git
+
+C'est mon premier repo git !
```

C'est très utile pour retrouver, après coup, les changements que vous aviez apportés à votre projet.
En cas de problème, vous pouvez voir exactement ce qui s'est passé au cours du temps.

### Autres commandes

Il y a encore beaucoup de choses que nous pourrions dire sur l'utilisation de git en local.
Nous vous encourageons à consulter la documentation pour plus d'informations.

Voici quelques commandes communes, que vous pouvez aller explorer :

* `git checkout -- .` : rétablit le working tree et l'index au statut du dernier commit (*danger*, mais très utile si vous réalisez que vous êtes parti-e dans la mauvaise direction depuis votre dernier commit).
* `git diff --cached` : montre les différences de l'index par rapport au head.
* `git revert <commit-sha>` : crée un commit "inverse" d'un commit donné, pour "annuler" ses changements.

## Synchronisation avec GitHub

**Rappel : tout comme git, ceci n'est pas imposé par le cours.
Si vous ne voulez vraiment pas utiliser GitHub, on ne va pas vous y forcer.**

Voyons maintenant comment vous pouvez synchroniser vos projets git avec un repo distant.
Cela vous permettra aussi, en deux étapes, de synchroniser vos projets entre deux ordinateurs que vous utilisez.
Par exemple, votre ordinateur personnel et les machines de l'EPFL.

[GitHub](https://github.com/) est un site très populaire de partage de projets *via* git.
La plupart des projets *open source* du monde informatique sont sur GitHub.
C'est ajourd'hui devenu la place centrale de développement collaboratif, en tous cas dans le monde open source.
Son concurrent principal est [GitLab](https://about.gitlab.com/).

Vous aurez besoin de d'un compte (gratuit) sur GitHub.
Ce compte vous permettra de créer des repos, privés ou publics.
Vous pourrez ensuite synchroniser vos repos locaux avec vos repos "remote".

### Créer votre compte

Suivez les instructions pour [créer un nouveau compte GitHub personnel](https://docs.github.com/fr/get-started/start-your-journey/creating-an-account-on-github#signing-up-for-a-new-personal-account).
Assurez-vous de choisir le plan "Free".

Vous devriez utiliser la même adresse e-mail pour votre compte GitHub que pour git en local.
Si ce n'est pas le cas, vous devriez au moins ajouter l'adresse e-mail de git en local comme adresse secondaire dans votre compte GitHub.
C'est ainsi que GitHub sait quels commits ont été créés par vous (que ce soit dans vos repos, ou partout ailleurs).

### Configurer vos clefs SSH

La documentation GitHub voudra vous diriger vers l'utilisation de ses propres outils pour éditer des repos et synchroniser en local (GitHub CLI, GitHub Desktop).
Ce ne sont pas ces outils que nous recommendons dans le cadre de ce cours.
Nous allons utiliser l'interface git originale pour cela.

Il faut donc que git ait la permission d'accéder à vos repos remote sur GitHub.
La méthode la plus répandue pour cela, avec git, est d'utiliser une paire de clefs SSH.
SSH est basé sur un système de chiffrement asymétrique, avec une clef publique et une clef privée.

> Eh oui ! C'est bien le type de systèmes que nous avons étudié au premier semestre avec RSA.

Ici, nous vous renvoyons vers la [documentation officielle de GitHub sur SSH](https://docs.github.com/fr/authentication/connecting-to-github-with-ssh), qui est vraiment très bien.
En particulier, vous devrez suivre les étapes suivantes :

* [Génération d'une nouvelle clé SSH et ajout de celle-ci à ssh-agent](https://docs.github.com/fr/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) (excepté la dernière section sur l'usage d'une clef de sécurité matérielle)
* [Ajout d'une nouvelle clé SSH à votre compte GitHub](https://docs.github.com/fr/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
* [Test de votre connexion SSH](https://docs.github.com/fr/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection)
* [Utilisation des phrases secrètes de clé SSH](https://docs.github.com/fr/authentication/connecting-to-github-with-ssh/working-with-ssh-key-passphrases)

#### Windows et Putty

Windows ne contient pas de client SSH par défaut.
C'est pourquoi git bash vous en fournit un aussi.

Cependant, beaucoup d'utilisateurs Windows préfèrent utiliser [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) à la place.
À vous de voir si vous voulez explorer cette option.
Si oui, consultez par exemple [ce document sur l'utilisation de Putty avec GitHub sur Windows](https://gist.github.com/jbrown123/16ede76b87aab1491c6b1dc809a4e03c).

#### Reconfiguration sur un autre ordinateur

Si vous utilisez plusieurs ordinateurs *auxquels vous faites confiance*, il n'est pas nécessaire de gérérer des clefs SSH pour chacun d'entre eux.
Vous pouvez copier votre clef privée d'un ordinateur à l'autre (de préférence *via* un canal sécurisé).
Assurez-vous de ne faire cela qu'avec des clefs protégées par des *phrases secrètes* (dernière étape ci-dessus).

### Repo remote et git push

Créez maintenant un repo GitHub pour `tuto-git`.

1. Ouvrez la page [Create a new repository](https://github.com/new).
2. Indiquez `tuto-git` dans "Repository name".
3. Sélectionnez "Private".
4. Laissez le reste vide/none. Nous allons connecter le repo que nous avons déjà commencé en local.
5. Validez en cliquant sur le bouton "Create repository".

**Important** : tous les repos que vous créez pour ce cours doivent être privés.
C'est particulièrement important pour le repo que vous utiliserez pour votre projet principal.

Sur la page qui s'affiche, dans la grande boîte bleue "Quick setup" :

1. Sélectionnez SSH.
2. Copiez l'URL indiquée. Elle devrait ressembler à `git@github.com:votre-compte/tuto-git.git`

Dans le terminal (toujours dans le dossier du projet `tuto-git`), entrez :

```
$ git remote add origin git@github.com:votre-compte/tuto-git.git
$ git push --set-upstream origin main
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (6/6), 561 bytes | 280.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:votre-compte/tuto-git.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

`git remote add` a ajouté le repo remote sous le nom `origin`.
`origin` est par convention le nom donné au remote principal d'un repo (il est possible d'en avoir plusieurs).

`git push` a ensuite envoyé le contenu de votre branche `main` vers le repo `origin`.
Au passage, l'argument `--set-upstream` (`-u` en version courte) configure `main` pour qu'elle sache qu'elle correspond à `origin/main`.

Vous pouvez maintenant rafraîchir la page GitHub, et vous devriez voir le contenu de votre projet.

Si vous créez de nouveaux commits, vous pouvez les envoyer sur le repo remote avec `git push` sans argument supplémentaire.

### Travailler avec le même repo sur un autre ordinateur

Si vous n'avez pas de second ordinateur à disposition pour faire ce test, vous pouvez aussi utiliser un autre dossier.

Dans la précédente section, nous avons connecté un nouveau repo remote à un repo local existant.
L'opération, qui consiste à créer une version locale d'un repo remote, s'appelle un *clone*.

Vous aurez besoin de la même URL de connexion `git@...`.
La commande `git clone` clone un repo remote dans un nouveau dossier local :

```
projets$ git clone git@github.com:votre-compte/tuto-git.git nouveau-dossier-tuto-git
Cloning into 'nouveau-dossier-tuto-git'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (6/6), done.

$ cd nouveau-dossier-tuto-git/

$ git log --oneline
6feba6e (HEAD -> main, origin/main, origin/HEAD) Add one section.
095ac8c Initial commit.
```

On peut voir que nous avons toute l'historique à disposition.

Créez un nouveau commit dans ce clone, par exemple en rajoutant une "seconde section" au fichier `README.md` :

```diff
 # Tutoriel git

 C'est mon premier repo git !

 ## Une section

 Un paragraphe.
+
+## Deuxième section
+
+Un autre paragraphe.
```

Les commandes suivantes vont ajouter les changements à l'index, en faire un commit, puis l'envoyer sur le remote :

```
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")

$ git add -u

$ git commit -m "Seconde section."
[main a57d2c3] Seconde section.
 1 file changed, 4 insertions(+)

$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 347 bytes | 347.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:sjrd/tuto-git.git
   6feba6e..a57d2c3  main -> main
```

Vous pouvez voir les nouveaux changements sur le site GitHub.

Pour les récupérer sur le premier ordinateur (ou le premier dossier), utilisez la commande `git pull` :

```
# dans le premier repo local
$ git pull
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0 (from 0)
Unpacking objects: 100% (3/3), 327 bytes | 18.00 KiB/s, done.
From github.com:sjrd/tuto-git
   6feba6e..a57d2c3  main       -> origin/main
Updating 6feba6e..a57d2c3
Fast-forward
 README.md | 4 ++++
 1 file changed, 4 insertions(+)
```

Vous avez désormais les outils en main pour synchroniser votre travail entre deux ordinateurs.
La semaine prochaine, nous verrons ce qui se passe lorsque plusieurs personnes travaillent en même temps sur différents clones d'un repo.
Nous examinerons le concept de *branche*, et comment git aide à *fusionner* différentes branches.

### Pour aller plus loin

* [Documentation de GitHub sur l'usage de git](https://docs.github.com/fr/get-started/using-git)
* [Lisez le livre "Pro Git"](https://git-scm.com/book/en/v2), qui est la référence ultime à propos de git
