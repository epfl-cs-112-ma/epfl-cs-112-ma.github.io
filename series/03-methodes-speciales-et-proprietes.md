---
title: "Série 3 : Méthodes spéciales et propriétés"
layout: default
use_mathjax: true
---

# Série 3 : Méthodes spéciales et propriétés

[Solutions](https://github.com/epfl-cs-112-ma/solutions-serie-03)

Cette série a pour objectif de pratiquer l'usage des principales méthodes spéciales, ainsi que des propriétés.

Avant de commencer cette série, il est prévu que vous ayez suivi les deux tutoriels de cette semaine :

* [Méthodes spéciales](/tutoriels/methodes-speciales.html)
* [Propriétés](/tutoriels/proprietes.html)

Nous vous recommandons de créer [un nouveau projet Python](/tutoriels/quick-projet-setup.html) pour chaque série d'exercices.
Cela vous permettra d'isoler plus facilement le contenu des différentes semaines.

Cette série contient 3 exercices :

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Mesures de distances avec unités

On souhaite définir une classe `Distance` qui représente une mesure de distance.
Cette mesure doit pouvoir être exprimée dans différentes unités, au minimum `km`, `m` et `mm`.
Une instance de `Distance` doit donc stocker deux valeurs : une unité, et une valeur flottante exprimée dans cette unité.

On requiert au moins les capacités suivantes :

* Construire une distance avec une valeur et son unité.
* Les instances de `Distance` doivent être immuables.
* Demander la valeur et l'unité.
* Afficher des distances.
* Demander la valeur convertie dans une unité donnée.
* Effectuer les opérations standard entre mesures de distances et/ou nombres sans dimension.

On ne vous demande **pas** de représenter d'autres mesures que les distances.

**Concevez** l'interface de la classe `Distance` pour qu'elle réponde aux exigences ci-dessus.
Ensuite, **implémentez** et **testez** `Distance`.

**Questionnement** : si `Distance` devait être muable, que changeriez-vous ?
Pensez-vous qu'il soit mieux d'avoir une class muable ou immuable, dans ce cas ?

## Liste chaînée immuable

Au semestre passé, nous avons introduit une représentation du concept abstrait de "liste" que nous avons nommée *liste chaînée*.
Cet exercice vous propose de concevoir et d'implémenter une telle liste chaînée immuable.

Pour rappel, au moins en interne, une liste chaînée immuable est

* soit une liste vide, que l'on peut représenter par `None` ;
* soit une paire (head, tail) où `head` est le premier élément de la liste, et `tail` est le reste de la liste, sous forme de liste.

On emballera cette représentation interne dans une classe `LinkedList`, qui exposera l'interface suivante :

* le constructeur crée une liste vide ;
* la propriété `is_empty` retourne `True` si et seulement la liste est vide ;
* les propriétés `head` et `tail` retournent le premier élément et une `LinkedList` des éléments 2 et suivants, respectivement.
* la méthode `prepend(x)` retourne une nouvelle liste composée de `x` suivi des éléments de la liste `self`
* il doit être possible d'utiliser `len(linked_list)` et `linked_list[i]` de manière adéquate.

On supposera qu'une `LinkedList` ne contient que des `int`.

**Élaborez** l'interface de `LinkedList` à partir de la définition informelle ci-dessus.
**Implémentez** et **testez** `LinkedList`.

Vous pourriez avoir besoin de définir une ou plusieurs "classes annexes".
Si c'est le cas, elles ne doivent pas apparaître dans l'interface publique de `LinkedList`.

Vous vous sentez **experte ou expert** ?
Ou peut-être revenez-vous à cette exercice en **fin de semestre** ?
Dans ce cas, faites en sorte que `LinkedList` puisse contenir n'importe quel type de données, tout comme la `list[T]` de Python.
