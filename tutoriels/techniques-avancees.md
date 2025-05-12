---
title: Techniques avancées
layout: default
use_mathjax: true
---

# Techniques avancées

Nous voilà déjà au dernier chapitre de ce cours.
Je vous propose ici une collection disparate de quelques techniques avancées, qui pourront vous servir lors de vos développements.
Ne cherchez pas le lien entre elles ; il n'y en a pas.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Transtypage et `Any`

Ou : **Quand les types statiques ne suffisent pas**

Les types statiques nous aident à structurer nos programmes.
Ils permettent de déléguer au compilateur une partie du travail, en lui laissant prouver certaines propriétés à propos de nos programmes.
Par exemple, que nous ne rencontrerons jamais d'`AttributeError` (accéder à un champs ou méthode qui n'existe pas), ou que nous gérons bien tous les cas possibles d'un [match exhaustif](./pattern-matching.html#exhaustivité-et-le-type-never).

Cependant, parfois, la complexité d'une portion de programme dépasse ce que l'on peut exprimer avec le système de types de notre langage.
Ou alors, c'est juste que nous est trop mauvais pour trouver la bonne solution.

Dans un cas comme dans l'autre, on peut alors avoir recours au *transtypage* (*cast*) et/ou au type `Any`.
Ces deux outils permettent d'enfreindre délibérément les règles du système de type.
De passer au travers des murs, en quelque sorte.

De ces deux techniques, le transtypage reste la plus cadrée.
C'est aussi la technique que vous retrouverez dans (presque) tous les langages que vous aurez à manipuler.
Le type `Any` est plus puissant, mais donc encore plus dangereux.

### Transtypage

Le transtypage, du verbe transtyper, mais souvent appelé *cast* même en français, consiste à dire au type checker : crois-moi, cette variable est de type `T`.
On l'exploite comme ceci :

```python
from typing import cast

def foo() -> object:
    return 21

def main() -> None:
    x = foo()        # x: object
    print(x * 2)     # Unsupported operand types for * ("object" and "int")
    y = cast(int, x) # Mypy and Python blindly trust us here, and y: int
    print(y * 2)     # OK
```

Il est important de comprendre que le type checker (Mypy) *et* l'exécution (Python) nous font totalement confiance.
*Aucune* vérification n'est faite qu'on n'a pas menti.
Cela veut dire que, si la variable `x` n'était en fait *pas* un `int`, on mettrait dans `y: int` son contenu, sans se poser de question.
Les erreurs qui en découleraient vont se produire plus tard, potentiellement loin de l'endroit où on a fait un appel à `cast`.

Par exemple, on peut facilement exploiter `cast` pour causer une `AttributeError` "à distance" :

```python
from typing import Final, cast

class Foo:
    foo: Final[int]

    def __init__(self, foo: int):
        self.foo = foo

class Bar:
    foo: str

    def __init__(self, foo: str):
        self.foo = foo

def make_foo() -> object:
    return Foo(21)

def make_bar() -> Bar:
    foo = make_foo()
    # return foo # Incompatible return value type (got "object", expected "Bar")
    # Ah, I'm sure it's fine:
    return cast(Bar, foo)

def main() -> None:
    # Mypy is perfectly happy here
    bar: Bar = make_bar()
    x: str = bar.foo
    # even Python, at run-time, is happy until the following line
    print(x.lower()) # AttributeError: 'int' object has no attribute 'lower'
```

Pire, `cast` peut être utilisé pour contourner certaines protections, telles que celles offertes par `Final`

```python
def main() -> None:
    foo: Foo = Foo(21)
    # foo.foo = 45 # Mypy: Cannot assign to final attribute "foo"
    # Let's do it anyway
    bar = cast(Bar, foo)
    bar.foo = "yolo"
    print(foo.foo) # "yolo"
    print(foo.foo // 2) # Run-time: TypeError: unsupported operand type(s) for //: 'str' and 'int'
```

En fonction des langages, et de ce que vous transtypez et quand, vous pourrez obtenir des résultats plus ou moins aberrants.
En Python, comme on peut le voir ci-dessus, on peut se retrouver avec n'importe quoi n'importe où.
En C++, c'est pire encore !
On peut corrompre la mémoire et créer de réelles failles de sécurité.
Dans des langages plus stricts, comme Java, vous finirez par tomber sur une `ClassCastException`, d'une manière ou d'une autre.
Peu importent les symptômes, le bug vient bien du `cast` incorrect, qui pouvait se trouver bien longtemps auparavant.

Dans presque toutes les situations, l'utilisation de meilleurs types vous évitera d'avoir recours au transtypage.
Sinon, voyez si un pattern match n'est pas plus adapté à votre situation.

Si vraiment vous avez besoin de `cast`, utilisez-le, avec en commentaires :

* pourquoi ce que vous faites est correct ;
* pourquoi vous n'avez pas pu (ou voulu) faire autrement.

Malgré tous ces défauts, le transtypage reste contrôlé : il y a *un* endroit où le type checker vous fait confiance.
Un endroit où vous devez réfléchir plus intensément, et commenter plus abondamment, pour garantir que votre programme reste correct.
C'est aussi un seul endroit que vos collègues double-checkeront quand ils ou elles reliront votre code.

Le transtypage sera votre outil si, au sein d'un programme bien typé par ailleurs, et utilisant des bibliothèques avec de bons types, il y a un endroit bien spécifique où vous n'avez pas réussi à convaincre le type checker (soit parce que le système de types est trop limité, soit parce que vous n'avez pas trouvé de meilleure solution).

### Type `Any`

Le type `Any` monte encore un cran au-dessus sur l'échelle de l'insécurité.
On peut assigner n'importe quoi à une variable de type `Any` (tout comme `object`, jusque là pas de problème), et on peut *utiliser* une variable de ce type *n'importe comment* (là on a des problèmes).

Par exemple, on peut lire et écrire des attributs quelconques d'un objet, sans savoir s'ils existent.
