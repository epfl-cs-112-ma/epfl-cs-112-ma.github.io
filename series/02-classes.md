---
title: "Série 2 : Classes et méthodes"
layout: default
use_mathjax: true
---

# Série 2 : Classes et méthodes

[Solutions](https://github.com/epfl-cs-112-ma/solutions-serie-02)

Dans cette seconde série d'exercices, nous vous proposons d'explorer les concepts de classes et méthodes.
Nous attendons de vous que vous mettiez en œuvre les notions d'interface et d'encapsulation de manière adéquate.

Avant de commencer cette série, il est prévu que vous ayez suivi le tutoriel correspondant :

* [Tutoriel Classes et méthodes](/tutoriels/classes.html)

Le tutoriel sur les branches git n'est pas nécessaire à cette série d'exercices.
Il est prévu pour votre projet.

Nous vous recommandons de créer [un nouveau projet Python](/tutoriels/quick-projet-setup.html) pour chaque série d'exercices.
Cela vous permettra d'isoler plus facilement le contenu des différentes semaines.

Cette série contient 3 exercices :

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Disque dans le plan

On s'intéresse ici à concevoir une classe `Disk` représentant un disque dans le plan.
Un disque est caractérisé par un centre et un rayon $> 0$.

N'oubliez pas d'écrire des tests pour tout ce que vous définissez.

> Vous pourriez obtenir un avertissement du type *"Disk" is not defined (Pylance)* (ou une `NameError` à l'exécution) sur l'usage de `Disk` comme type au sein de la classe `Disk` elle-même.
> Si c'est le cas, ajoutez la ligne suivante *au tout début* du fichier :
>
> ```python
> from __future__ import annotations
> ```
>
> De manière générale, nous aurons souvent besoin de cette ligne, étant donné notre usage intensif des types en Python.
> Vous pouvez prendre l'habitude de l'inclure systématiquement.

La constante $\pi$ peut être obtenue depuis le module `math` :

```python
import math

print(math.pi)
```

De même, la racine carrée de `x` s'obtient avec `math.sqrt(x)`.

### Définition élémentaire

Définissez cette première classe.
Il devrait être possible de créer n'importe quel disque valide sans *modifier* ses attributs après l'avoir construit.

Comment modélisez-vous le centre ?

### Aire

Définissez une méthode `area` permettant d'obtenir l'aire d'un disque.

### Appartenance d'un point au disque

Définissez une méthode `contains_point` permettant de tester si un point du plan est sur le disque.
Pour rappel, par définition, un point $p$ fait partie d'un disque $(c, r)$ (fermé) si et seulement si $d \leq r$ où $d$ est la distance entre $c$ et $p$.

### Encapsulation

Questionnement de votre design :

* Est-il possible de *construire* un disque invalide (avec $r \leq 0$) ?
* Est-il possible de *modifier* un disque valide en un disque invalide ?

Si vous avez répondu "oui" à l'une ou deux de ces questions, révisez votre implémentation et/ou conception.

**Conseil** : Il est tout à fait normal de commencer par une conception qui se préoccupe uniquement du "happy path", c'est-à-dire des cas valides.
Il faut cependant toujours se poser la question des cas problématiques avant de considérer une conception comme terminée.
Sur papier, lors des examens, vous aurez sans doute intérêt à anticiper un peu plus.
Cependant il est difficile d'anticiper si vous n'avez pas d'abord acquis l'expérience en y réfléchissant après coup.

## Triangle dans l'espace

Dans cet exercice, nous modélisons un triangle dans l'espace à 3 dimensions.
Un triangle est caractérisé par ses trois sommets.
Chaque sommet est un point de l'espace.

**Concevez et implémentez** les classes `Point3D` et `Triangle`.

La classe `Triangle` doit proposer au moins deux méthodes :

* `is_isosceles` teste si le triangle est isocèle.
  Pour rappel, un triangle est isocèle s'il possède au moins 2 côtés de même longueur.
* `perimeter` donne le périmètre du triangle.

Pour information, le triangle $\langle (0, 0, 1), (-3, 4, 5), (3, 4, -3) \rangle$ est isocèle.

## Le petit train

Dans cet exercice, nous cherchons à modéliser un petit train dont la progression est contrôlée par des feux de signalisation.
Le circuit de rails est linéaire, mais est ponctué de passages à niveaux qui sont protégés par des feux.
Nous ne modélisons pas les passages en tant que tels ; seulement les feux.

Le circuit est décomposé en *tronçons* (*segment*).
Chaque tronçon connaît le tronçon suivant, à moins qu'il ne soit le terminus.
*Certains* tronçons sont attachés à un feu.

Le train est à tout moment sur un tronçon donné.
On ne représentent pas les états intermédiaires lorsqu'on est physiquement en train de passer d'un tronçon à l'autre.
Il peut avancer au tronçon suivant si ce n'est pas le terminus, et si l'une des deux conditions suivantes est respectée :

* le tronçon actuel n'est attaché à aucun feu, ou
* le feu attaché est vert.

Les concepts en jeu sont les suivants :

* Un feu de signalisation (`Light`) qui peut être dans l'état rouge ou vert.
* Un tronçon de chemin de fer (`Segment`) qui connaît le tronçon suivant (à moins que ce ne soit le dernier) et l'éventuel feu attaché.
* Le train (`Train`) qui connaît le tronçon sur lequel il se trouve.

Les feux sont manipulés par un opérateur du réseau, et le train est manipulé par une conductrice.
Ces entités ne sont pas modélisées ici.
Ce sont vos tests et/ou une fonction `main` qui vont directement jouer ces rôles.

L'interface donnée à la conductrice doit au moins avoir les fonctionnalités suivantes :

* Demander si le train peut avancer au prochain tronçon.
* Avancer au prochain tronçon.

**Modélisez et implémentez** tout ce petit monde.
Commencez par les feux, puis les tronçons, puis le train.
Assurez-vous d'utiliser les concepts d'interface et encapsulation de manière adéquate.
Testez.
