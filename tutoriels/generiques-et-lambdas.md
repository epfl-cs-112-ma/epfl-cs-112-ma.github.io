---
title: Génériques et lambdas
layout: default
use_mathjax: true
---

# Génériques et lambdas

D'une part, vous savez définir vos propres classes, comme une `class A(B):`.
Vous savez aussi que chaque classe définit un *type* associé, ici `A`.
De plus, on dérive des relations de *sous-typage* (`A <: B`) à partir des relations d'héritage (`A` étend `B`).

D'autre part, vous savez utiliser plusieurs structures de données, dont `list[T]`, `dict[K, V]`, etc.
Vous aurez remarqué que le code qui *utilise* ces structures peut *choisir* le type d'éléments qu'elles contiennent.
Vous pouvez avoir des `list[int]` ou des `list[A]`, par exemple.
Les classes comme `list` et `dict` sont dites *génériques*.
De même, des fonctions telles que `sorted` sont génériques : on peut les appeler avec des `list[T]` pour n'importe quel type `T`.

Vous êtes-vous demandé si *vous* pouviez aussi *définir* des classes ou des fonctions génériques ?
La réponse est évidemment "oui", et c'est ce que nous étudions aujourd'hui.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Fonctions génériques

### Fonction `reverse`

Commençons par un exemple canonique de fonction générique : la fonction `reverse`, qui prend une liste et en renvoie une nouvelle dont les éléments sont dans l'ordre inverse.
Si nous voulons qu'elle fonctionne avec des `list[int]`, nous pouvons l'implémenter comme suit :

```python
def reverse_int(xs: list[int]) -> list[int]:
    result: list[int] = []
    for i in range(len(xs) - 1, -1, -1):
        result.append(xs[i])
    return result
```

avec son test associé :

```python
def test_reverse_int() -> None:
    assert reverse_int([2, 3, 5, 7]) == [7, 5, 3, 2]
    assert reverse_int([]) == []
    assert reverse_int([11]) == [11]

    # make sure that the input list is not modified
    xs = [13, 17, 23]
    assert reverse_int(xs) == [23, 17, 13]
    assert xs == [13, 17, 23]
```

Notez au passage qu'il y a plus de lignes de tests que de la fonction elle-même.
C'est très courant.

Maintenant, si nous voulons aussi une fonction pour renverser des `list[str]`, nous devons définir une nouvelle fonction :

```python
def reverse_str(xs: list[str]) -> list[str]:
    result: list[str] = []
    for i in range(len(xs) - 1, -1, -1):
        result.append(xs[i])
    return result
```

Remarquez que, en dehors du nom de la fonction, seules les annotations de types ont changé.
L'implémentation est entièrement la même !
Et il en faudra une nouvelle pour *chaque* type d'élément sur lesquels on veut travailler.

C'est évidemment inacceptable.
On souhaite définir une seule fonction `reverse` qui peut fonctionner avec n'importe quel type.

D'un point de vue mathématique, on peut voir `reverse_int` comme une fonction $\texttt{list[int]} \rightarrow \texttt{list[int]}$.
On pourrait dire que $\texttt{list[int]} \rightarrow \texttt{list[int]}$ est le *type* de `reverse_int` (c'est un type de fonction).
Ce qu'on voudrait, c'est une *famille* de fonctions $\texttt{reverse}_T$, *paramétrable* en $T$, dont les types respectifs seraient $\texttt{list[}T\texttt{]} \rightarrow \texttt{list[}T\texttt{]}$.
Dit avec encore moins de mots et encore plus de symboles mathématiques, on veut :

$$ \forall T. \texttt{reverse}_T: \texttt{list[}T\texttt{]} \rightarrow \texttt{list[}T\texttt{]} $$

Malheureusement, on ne fait pas des maths, ici.
On ne peut pas se permettre de définir une "infinité" de fonctions.
Cependant, on peut déplacer intelligemment le $\forall T$ de cette formule pour qu'il n'y ait qu'une fonction `reverse`, avec un type un peu étrange :

$$ \texttt{reverse}: \forall T. \texttt{list[}T\texttt{]} \rightarrow \texttt{list[}T\texttt{]} $$

Il se trouve que ça, nous pouvons réellement le définir en Python (et dans presque tous les langages modernes) :

```python
def reverse[T](xs: list[T]) -> list[T]:
    result: list[T] = []
    for i in range(len(xs) - 1, -1, -1):
        result.append(xs[i])
    return result
```

Le nouvel élément de syntaxe est que nous commençons avec `def reverse[T]`.
Cela fait de `reverse` une fonction *générique en $T$*.
Tout comme `xs` est un *paramètre* (valeur) de `reverse`, on appelle $T$ un *paramètre de type* (*type parameter*).
Dans la signature (les types des paramètres et de résultat) et l'implémentation de `reverse`, on peut utiliser $T$ comme n'importe quel type.
On en parle alors aussi comme d'une *variable de type* (*type variable*).

Nous pouvons appeler cette méthode en lui donnant en arguments, soit une `list[int]`, soit une `list[str]`.
Plus généralement, on peut lui donner une `list[U]` pour n'importe quel `U`.
En fonction du type de liste que l'on donne en paramètre, on reçoit le type approprié en retour :

```python
result1: list[int] = reverse([2, 3])
result2: list[str] = reverse(["hello", "world"])
```

On peut étendre un peu notre test pour vérifier tout cela :

```python
def test_reverse() -> None:
    assert reverse([2, 3, 5, 7]) == [7, 5, 3, 2]
    assert reverse([]) == []
    assert reverse([11]) == [11]

    assert reverse(["foo", "bar"]) == ["bar", "foo"]

    # assert that we get the types we hope for
    result1: list[int] = reverse([2, 3])
    assert result1 == [3, 2]
    result2: list[str] = reverse(["hello", "world"])
    assert result2 == ["world", "hello"]

    # make sure that the input list is not modified
    xs = [13, 17, 23]
    assert reverse(xs) == [23, 17, 13]
    assert xs == [13, 17, 23]
```

### Polymorphisme paramétrique

On peut considérer que la fonction `reverse` ci-dessus prend différentes *formes* en fonction de comment on l'appelle.
Tantôt elle se comporte comme une $\texttt{list[int]} \rightarrow \texttt{list[int]}$, tantôt comme une $\texttt{list[str]} \rightarrow \texttt{list[str]}$.
En ce sens, elle fait preuve de *polymorphisme*.

Ce n'est toutefois pas la même catégorie de polymorphisme que ce que nous avons vu avec le sous-typage.
Dans ce cadre-là, on parlait de *polymorphisme par sous-typage*.
Ici, nous pouvons *paramétrer* `reverse` pour lui faire prendre différentes formes.
On parle donc de *polymorphisme paramétrique*.

Un autre nom qui est parfois donné au même concept est celui de *polymorphisme universel*.
Ce nom fait référence au *quantificateur universel* $\forall T$ associé au type `reverse`.

De manière moins "savante", on fait aussi souvent référence au même concept sous le nom de "génériques".
C'est ce nom que vous verrez le plus souvent dans de la documentation à propos de Python.

## Lambdas

### Limites de l'universalité

Nous avons pu définir une fonction `reverse` générique.
À l'intérieur, nous manipulons des valeurs de type `T`.
Par exemple, `xs[i]` a le type `T`, puisque c'est un élément d'une `list[T]`.
On peut mettre en évidence ce type en ajoutant (pour l'illustration pédagogique) une variable intermédiaire :

```python
def reverse[T](xs: list[T]) -> list[T]:
    result: list[T] = []
    for i in range(len(xs) - 1, -1, -1):
        element: T = xs[i]
        result.append(element)
    return result
```

Vous aurez peut-être remarqué qu'on ne fait pas grand chose *avec* ces valeurs de type `T`.
Tout ce qu'on fait dans `reverse`, c'est les "déplacer".
On peut, en fait, pas faire grand chose d'autre avec.
On ne peut pas appeler de méthode dessus, par exemple.

C'est le prix à payer pour *l'universalité* de ce type de polymorphisme.
Puisque `reverse` doit fonctionner *pout tout type* $T$, nous ne pouvons utiliser que les opérations qui sont valables avec n'importe quel type.
En dehors de `==`, il n'y a pas grand chose.

Essayons de définir une autre fonction, qui apparemment devrait pouvoir être générique : filtrer
