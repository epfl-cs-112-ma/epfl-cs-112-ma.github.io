---
title: Itérateurs et générateurs
layout: default
use_mathjax: true
---

# Itérateurs et générateurs

Ce tutoriel introduit les itérateurs, l'abstraction sur laquelle sont bâties toutes les *comprehensions* de Python.
Nous montrons trois méthodes d'écriture d'itérateurs personnalisés : à la main, avec des *expressions generateur* (*generator expression*) et avec des fonctions génératrices (*generators*).

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Itérateurs

Vous avez déjà utilisé des itérateurs, sans le savoir.
À chaque fois que vous utilisez une boucle `for` ou une comprehension, Python utilise un itérateur.
Par exemple, si vous écrivez :

```python
for x in my_list:
    print(x)
```

cela correspond au code suivant :

```python
iterator = iter(my_list)
try:
    while True:
        x = next(iterator)
        print(x)
except StopIteration:
    pass
```

La fonction builtin `iter(xs)` crée un objet *itérateur* (*iterator*) pour `xs`.
Un itérateur contient l'état nécessaire à itérer sur des éléments.
Concrètement, un itérateur doit définir une méthode spécial `__next__()` :

* si il y a encore au moins un élément à itérer, retourner l'élément suivant ;
* sinon, lancer une exception de type `StopIteration`.

Vous aurez deviné que la fonction builtin `next(iterator)` appelle en fait `iterator.__next__()`.

On pourrait par exemple écrire un itérateur personnalisé qui itère sur les nombres de `0` à `n`, comme ceci :

```python
class IntIterator:
    __n: Final[int]
    __i: int

    def __init__(self, n: int) -> None:
        self.__n = n
        self.__i = 0

    def __next__(self) -> int:
        if self.__i < self.__n:
            self.__i += 1
            return self.__i - 1
        else:
            raise StopIteration()
```

On pourrait alors itérer manuellement dessus, comme ceci :

```python
>>> it = IntIterator(5)
>>> next(it)
0
>>> next(it)
1
>>> next(it)
2
>>> next(it)
3
>>> next(it)
4
>>> next(it)
Traceback (most recent call last):
  File "<python-input-16>", line 1, in <module>
    next(it)
    ~~~~^^^^
  File "<python-input-9>", line 14, in __next__
    raise StopIteration()
StopIteration
```

Remarquez qu'un itérateur est intrinsèquement muable.
On veut qu'il renvoie un nouvel élément à chaque appel à `next()`, et qu'il finisse par lancer une exception de type `StopIteration`.

### Itérateurs et boucles `for`

Pour pouvoir en profiter dans une boucle `for`, il faut qu'un appel à `iter(xs)` renvoie une instance de `IntIterator`.
Encore une fois, la fonction builtin `iter` appelle la méthode spéciale `__iter__`.

```python
class IntRange:
    __n: Final[int]

    def __init__(self, n: int) -> None:
        self.__n = n

    def __iter__(self) -> IntIterator:
        return IntIterator(self.__n)

my_range = IntRange(5)
for x in my_range:
    print(x)
```

Notez que Mypy sait que `x` est un `int`, ici, puisque c'est le type d'élément renvoyé par `IntIterator.__next__()`.

Contrairement à `IntIterator`, qui est forcément muable, `IntRange` est immuable.
À chaque fois que l'on appelle `iter()`, on reçoit un nouvel `IntIterator`.
L'état de chaque `IntIterator` n'a aucun effet sur l'instance de `IntRange`.

Vous aurez sans doute remarqué que `IntRange(5)` ressemble étrangement au `range(5)` de Python, que vous utilisez depuis le début du semestre.
C'est en substance comment `range` est implémentée : comme une classe avec une méthode spéciale `__iter__`.

### `Iterable[T]` et `Iterator[T]`

Les instances de classes qui ont une méthode `__iter__` sont *itérables*.
La classe abstraite `collections.abc.Iterable[T]`, que vous connaissez déjà, représente exactement cette capacité.
Une classe abstraite `Iterator[T]` correspondante représente les iterateurs.
Pour bien faire, on devrait donc définir `IntIterator` et `IntRange` comme

```python
class IntIterator(Iterator[int]):
    ... # as before

class IntRange(Iterable[int]):
    ... # as before
```

### Itérateurs et comprehensions

Les boucles `for` ne sont pas les seules constructions de langage qui utilisent les itérateurs.
C'est sur le même mécanisme que sont bâties les list/set/dict comprehensions.
Dans le chapitre sur [les structures de données](./structures-de-donnees.html#list-comprehensions), nous avons vu que les comprehensions peuvent s'expliquer en termes de boucles `for`.
Mais puisque les boucles `for` fonctionnent avec des itérateurs, il en va de même des comprehensions.

Nous pouvons construire la liste des 5 premiers naturels pairs grâce à notre `IntRange` :

```python
print([i * 2 for i in IntRange(5)]) # [0, 2, 4, 6, 8]
```

## Générateurs

Les générateurs sont des moyens simples d'écrire des itérateurs.

### Generator expressions

On a souvent utilisé des list comprehensions pour donner filtrer ou transformer des collections, avant de les donner à d'autres fonctions.
Par exemple, si l'on veut sommer les 5 premiers naturels pairs, on écrirait :

```python
sum([i * 2 for i in IntRange(5)]) # 20
```

Cependant, cela fait du travail inutile.
Ce code *alloue* une liste intermédiaire contenant les valeurs `[0, 2, 4, 6, 8]`, juste pour les sommer, avant de s'en débarrasser.
Bien que cela soit de complexité $\Theta(n)$, ce qui est acceptable, on paie un coût non nul en termes de performances brutes.

On pourrait éviter ce coût en performance en faisant la somme manuellement :

```python
result = 0
for i in IntRange(5):
    result += (i * 2)
```

mais c'est très insatisfaisant.
Le code avec `sum()` et la list comprehension est plus clair.
Notamment, il ne manipule pas directement d'état muable (`result`).

On peut obtenir les bonnes performances du second code, avec l'élégance de `sum`.
Il suffit pour cela d'utiliser une *generator expression*.
Attention, la différence est subtile :

```python
sum(i * 2 for i in IntRange(5)) # 20
```

La seule différence est qu'on a retiré les `[]` !
Entre parenthèses (et donc notamment dans les arguments d'un appel de fonction), une comprehension est une *generator expression*.
Elle ne crée pas une liste intermédiaire avec tous les éléments.
À la place, elle crée un objet itérateur (un *générateur*) dont l'itérateur délègue à l'itérateur de `IntRange(5)` et multiplie chaque élément par 2.

On aurait pu écrire ce générateur à la main comme suit :

```python
class DoubleIterator(Iterator[int]):
    __underlying: Iterator[int]

    def __init__(self, underlying: Iterator[int]) -> None:
        self.__underlying = underlying

    def __next__(self) -> int:
        underlying_next = next(self.__underlying)
        return underlying_next * 2

class DoubleGenerator(Iterable[int]):
    __underlying: Iterator[int]

    def __init__(self, underlying: Iterator[int]) -> None:
        self.__underlying = underlying

    def __iter__(self) -> Iterator[int]:
        return DoubleIterator(self.__underlying)

sum(DoubleGenerator(iter(IntRange(5))))
```

C'est un joli paquet de code que nous évitons grâce à la generator expression !

Remarquez que `DoubleIterator` n'a pas l'air de déclencher de `StopIteration` pour s'arrêter.
Lorsque `next(self.__underlying)` lance une `StopIteration`, `DoubleIterator.__next__` la propage implicitement, et donc se termine aussi.

Toutes les fonctions qui acceptent des `Iterable[T]` acceptent des generator expressions en argument.
C'est logique, puisque les générateurs étendent `Iterable[T]`.

Lorsque vous écrivez vos propres fonctions qui prennent des collections en paramètres, et que vous avez seulement besoin de pouvoir itérer dessus, vous devriez vous aussi demander des `Iterable[T]`.
Cela permettra au code qui appelle vos fonctions de leur donner des générateurs en argument, ce qui sera plus efficace que de construire des listes intermédiaires.

### Fonctions génératrices

Une fonction génératrice ressemble beaucoup à une fonction normale, mais elle constitue implicitement un générateur.
On la distingue par la présence du mot-clef `yield` quelque part dans le corps de la fonction.

Si l'on revient à notre `IntRange`, nous aurions en fait pu l'écrire comme une fonction génératrice :

```python
def int_range(n: int) -> Iterator[int]:
    i = 0
    while i < n:
        yield i
        i += 1
```

que l'on peut utiliser comme avant :

```python
[i * 2 for i in int_range(5)] # [0, 2, 4, 6, 8]
```

Bien qu'`int_range` déclare renvoyer un `Iterator[int]`, elle ne contient pas de `return`.
La seule présence du `yield` change complètement le comportement de la fonction.
Lorsqu'on l'appelle, les choses suivantes se passent :

1. Elle crée un nouveau générateur qu'elle renvoie immédiatement (le corps de la fonction n'est donc *pas* exécuté au moment où on l'appelle).
2. La méthode `__iter__()` de ce générateur ne peut être appelée qu'une seule fois, et renvoie un itérateur.
3. La méthode `__next__()` de cet itérateur démarre le corps de la fonction.
   Lorsqu'elle rencontre un `yield`, cette méthode `__next__()` renvoie la valeur donnée au `yield` (donc comme si elle faisait `return i` ici).
4. La prochaine fois qu'elle est appelée, elle *continue* le corps de la fonction là où elle s'était interrompue.
   À nouveau, au prochain `yield`, elle renvoie la valeur.
5. Lorsque finalement le corps de la fonction se termine, la méthode `__next__()` déclenche une `StopIteration`.

Les fonctions génératrices sont très bizarres sur le plan technique, puisqu'elles s'exécutent "par petits bouts".
À chaque appel à la méthode `__next__()` de l'itérateur implicite, on avance un petit peu dans l'exécution du corps de la fonction.

Leur fonctionnement est cependant assez intuitif, si on ne regarde pas trop les détails.
Elles renvoient un itérateur qui va itérer sur les valeurs "renvoyées" par `yield`.

Si vous vous retrouvez à souvent faire la même boucle `for` imbriquées et/ou avec les mêmes `if`, une fonction génératrice peut grandement vous simplifier la vie.
Par exemple, imaginons que vous ayez souvent besoin d'itérer sur les naturels qui sont multiples de 2 ou de 3 (ou les deux).
Vous écririez alors souvent :

```python
for i in range(100):
    if i % 2 == 0 or i % 3 == 0:
        ... # do something with i
```

Vous pouvez vous simplifier la vie en définissant une fois une fonction génératrice :

```python
def multiples_of_2_or_3(upper_bound: int) -> Iterator[int]:
    for i in range(upper_bound):
        if i % 2 == 0 or i % 3 == 0:
            yield i
```

ce qui vous permettra de remplacer vos itérations habituelles par

```python
for i in multiples_of_2_or_3(100):
    ... # do something with i
```
