---
title: "Série 6 : Génériques et lambdas"
layout: default
use_mathjax: true
---

# Série 6 : Génériques et lambdas

[Solutions](https://github.com/epfl-cs-112-ma/solutions-serie-06)

Cette série a pour objectif de pratiquer la définition de méthodes et classes génériques, ainsi que les lambdas et fonctions d'ordre supérieur.

Avant de commencer cette série, il est prévu que vous ayez suivi le tutoriel de cette semaine :

* [Génériques et lambdas](/tutoriels/generiques-et-lambdas.html)

Nous vous recommandons de créer [un nouveau projet Python](/references/quick-projet-setup.html) pour chaque série d'exercices.
Cela vous permettra d'isoler plus facilement le contenu des différentes semaines.

Cette série contient 2 exercices :

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Manipulations de listes

Dans ce premier exercice, vous devez implémenter quelques fonctions qui manipulent des listes de manière générique.

### Insertion dans une liste triée

Écrivez une fonction `insert_sorted` qui prend en paramètres : une liste `xs: list[int]` triée par ordre croissant, et un élément `y: int`.
Elle doit renvoyer une nouvelle liste `zs: list[int]` qui satisfait les conditions suivantes :

* `len(zs) == len(xs) + 1`
* `zs` contient tous les éléments de `xs`, ainsi que l'élément `y` ; s'il y a des doublons, ils doivent être préservés
* `zs` est triée

La complexité de `insert_sorted` doit être $\Theta(n)$ où $n$ est la taille de la liste `xs`.
Testez votre fonction.

Généralisez ensuite cette fonction, en deux étapes.

D'abord, faites en sorte qu'on puisse personnaliser la fonction de comparaison : plutôt qu'utiliser systématiquement `a < b`, il faudra utiliser `lessThan(a, b)`.
Ajoutez un paramètre `lessThan` qui est une *fonction* $(\texttt{int}, \texttt{int}) \rightarrow \texttt{bool}$.
Son type Python est donc `Callable[[int, int], bool]`.
La liste `xs` est maintenant triée *selon la fonction de comparison `lessThan`*.
De même, la liste `zs` doit respecter ce tri.
Vérifiez que vos tests existants passent toujours lorsqu'on utilise la fonction `lambda a, b: a < b` en argument.
Ajoutez de nouveaux tests avec d'autres fonctions de comparaison.
Par exemple, quelle fonction de compaison devez-vous utiliser pour travailler avec une liste triée par ordre *décroissant* ?

Ensuite, faites en sorte qu'on puisse utiliser votre fonction avec n'importe quel type de liste.
Les listes `xs` et `zs` sont maintenant d'un type `list[T]` où `T` est générique.
Vérifiez que tous vos tests existants passent tels quels.
Ajoutez de nouveaux tests qui utilisent d'autres types de listes (par exemple, des chaînes de caractères).

### Compteur de fréquences générique

Dans la [série 5 sur les structures de données](./05-structures-de-donnees.html), vous avez écrit une fonction qui comptait les fréquences des mots dans un texte.
Vous allez maintenant en écrire une version générique, mais qui travaille directement sur des listes et renvoie directement un `dict` (il n'y a pas d'étape de découpe, de formattage, ni de mots à ignorer).

D'abord, identifiez le coeur de l'algorithme: écrivez une fonction `frequencies` qui prend en paramètre une liste `xs: list[str]` et qui renvoie un `dict[str, int]`.
Le dictionnaire doit associer à chaque chaîne présente au moins une fois dans `xs`, le nombre de fois qu'elle apparaît dans `xs`.
Testez votre fonction.

Ensuite, *généralisez* votre fonction pour qu'elle puisse travailler sur des listes de n'importe quel type.
Vérifiez que vos tests existants passent toujours, et écrivez de nouveaux tests avec d'autres types de données.

**Questionnement :** votre fonction peut-elle vraiment travailler avec *n'importe quel* type ?

## Arbre binaire générique

Dans la [série 1](./01-appropriation-de-python.html), vous avez travaillé avec des arbres binaires spécifiques.
Vous allez ici définir une structure similaire, mais générique et avec des propriétés un peu différentes.
Dans cette version, ce sont les *feuilles* qui portent des valeurs et ont un *poids*.
Les *branches* de l'arbre ne possèdent que deux sous-arbres, et ne portent pas de valeur elles-mêmes.

On ne peut donc pas la représenter par `Node | None` comme nous l'avions fait en semaine 1.
À la place, nous allons utiliser l'héritage et le polymorphisme.

Définissez une classe abstraite `Tree` qui représente un arbre binaire immuable.
Elle a deux sous-classes `Leaf` et `Branch`, représentant les deux possibilités pour un arbre.

* `Leaf` est une feuille de l'arbre, qui porte une valeur (`value`) de type `str` et un poids (`weight`) de type `int`.
* `Branch` est une branche de l'arbre qui possède deux sous-arbres `left` et `right`.

### Poids d'un arbre

Généralisez le poids pour tous les `Tree`.
Il doit être possible d'accéder au poids d'un `t: Tree` avec `t.weight`, que ce soit une feuille ou une branche.
Le poids d'une branche est, par définition, la *somme* des poids de ses sous-arbres.

Si vous possédez une `list[Tree]` triée par ordre croissant des poids, comment insérez-vous un nouveau `Tree` pour obtenir une liste toujours triée de la même façon ?

### Parcours d'un arbre

Écrivez une méthode `values` qui ne prend pas d'argument et renvoie un `set[str]`.
L'ensemble doit contenir les *valeurs* de toutes les feuilles de l'arbre.
Plus exactement, on définit `values` par induction :

* `leaf.values() = {leaf.values}`
* `branch.values() = branch.left.values() | branch.right.values()` (rappel : l'opérateur `|` est l'union d'ensembles)

(ceci est une *définition* ; votre implémentation peut être très différente)

### Arbre générique

Rendez votre arbre générique *en le type de valeur des feuilles*.
Pour un `Tree[T]`, la `value` des feuilles de l'arbre de l'arbre doit être de type `T`.

### Chemins

Un *chemin* est la séquence des `left/right` qu'il faut suivre, à partir d'un `tree: Tree[T]`, pour arriver à une feuille de cet arbre.
On peut représenter un chemin par une `list[Side]` où `Side` est un `Enum` avec les valeurs possible `Side.LEFT` et `Side.RIGHT`.

Écrivez une méthode `follow_path` qui prend un chemin en argument et renvoie la valeur `T` trouvée au bout du chemin.

### Valeurs avec leurs chemins

Écrivez une méthode `values_with_paths`, qui ne prend pas d'argument, et renvoie un dictionnaire.
Ce dictionnaire doit associer à chaque valeur trouvée dans une feuille de l'arbre, le chemin qui amène à cette feuille.

Commencez par écrire une *définition* inductive de `values_with_path`, similaire en structure à celle que nous vous avons donnée pour `values`.
Quel est le type de résultat de `values_with_paths` ?
