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

#### Exemple de transtypage

Prenons un exemple concret qui pourrait arriver dans la vraie vie.
Soient deux classes `A` et `B` *disjointes* (il n'y a pas de sous-classe commune).
On doit écrire une fonction qui prend une `Sequence[A] | Sequence[B]`, et qui doit faire quelque chose avec tous les `A` si c'est une `Sequence[A]`, et autre chose avec tous les `B` sinon.
Remarquez ce n'est *pas* équivalent à `Sequence[A | B]`.
Dans cette dernière, on peut mélanger des `A` et des `B`.
Dans `Sequence[A] | Sequence[B]`, il y a soit uniquement des `A`, soit uniquement des `B`.
Si la séquence on vide, on choisit de la traiter comme une `Sequence[B]`.

Normalement, on utiliserait un pattern match exhaustif pour distinguer des cas disjoints dans une union.
Ici, ce n'est pas possible, car on peut seulement tester le "premier niveau" : si c'est une `Sequence` ou pas, mais pas si c'est une `Sequence[A]`.
On va donc se reposer sur tester *un* élément de la séquence.
Si c'est un `A`, on sait que tous les autres sont des `A` aussi.
Mais le type checker ne le sait toujours pas, et c'est donc là que le transtypage intervient.

```python
from collections.abc import Sequence
from typing import cast

class A: pass
class B: pass

def do_something_with_as(s: Sequence[A]) -> None: pass
def do_something_with_bs(s: Sequence[B]) -> None: pass

def do_something(s: Sequence[A] | Sequence[B]) -> None:
    # if s is non-empty and its first element is an A
    if s and isinstance(s[0], A):
        # then we know they are all A's, although the type system cannot prove it
        do_something_with_as(cast(Sequence[A], s))
    else:
        # otherwise they are all B's (this includes the empty case)
        do_something_with_bs(cast(Sequence[B], s))
```

Il n'y a pas de quoi en tirer fierté, mais au moins ici un `cast` bien placé nous permet de convaincre le type system.
Remarquez qu'ici on peut humainement prouver que c'est correct (en supposant que `A` et `B` sont effectivement disjointes).
Il y a des propriétés à propos du type system qui peuvent être prouvées humainement, mais pas être prouvées par le système.

### Type `Any`

Le type `Any` monte encore un cran au-dessus sur l'échelle de l'insécurité.
On peut assigner n'importe quoi à une variable de type `Any` (tout comme `object`, jusque là pas de problème), et on peut *utiliser* une variable de ce type *n'importe comment* (là on a des problèmes).

Par exemple, on peut lire et écrire des attributs quelconques d'un objet, sans savoir s'ils existent.
De plus, `Any` est *viral* : toutes propriétés qu'on lit sur des `Any`, et tous les résultats de méthodes appelées sur des `Any`, sont eux-mêmes des `Any`.
On peut très vite polluer beaucoup de code avec des types `Any` qui ne sont absolument pas vérifiés.

```python
from typing import Any

class A:
    def foo(self) -> B:
        return B()

class B:
    def bar(self, y: int) -> int:
        return 42 + y

def main() -> None:
    a: Any = A()
    b = a.foo()
    r = b.bar(5)
    print(r * 2)
```

`b` et `r` sont tous deux des `Any` également dans cet exemple.
Ici, on a réfléchi intensément, et on a écrit du code qui fonctionne effectivement.
Mais on peut écrire plein de variation qui vont donner des résultats aberrants.

Quelques exemples (indépendants) qui provoquent tous des erreurs à l'exécution (ce ne sont pas des erreurs de Mypy) :

```python
# AttributeError: 'A' object has no attribute 'foobar'
b = a.foobar()

# TypeError: B.bar() takes 2 positional arguments but 3 were given
r = b.bar(5, 6)

# TypeError: unsupported operand type(s) for +: 'int' and 'str'
r = b.bar("hello")

# TypeError: object of type 'int' has no len()
r2: str = r
print(len(r2)) # erreur ici
```

`Any` est donc nettement plus dangereux qu'un `cast`.
Par sa nature virale, il faut réfléchir intensément à toutes les utilisations successives, pour vérifier que tout est correcte.
Avec le transtypage, il n'y a qu'un endroit à valider.
Il faut donc limiter le plus possible la zone dangereuse dans laquelle on manipule les `Any`.

Alors à quoi ça sert ?
Beaucoup de langages n'ont rien qui ressemble au type `Any` : C++, Java, Scala, C#, ont tous une forme de transtypage, mais pas de type équivalent à `Any`.
Les langages qui ont quelque chose de similaire incluent TypeScript, Hack, Typed Racket, Typed Closure.
Ces langages ont tous quelque chose en commun : **ils ont initialement été conçus sans système de types**.
C'est aussi le cas de Python.
Pour la plupart, le système de types ne fait même pas partie du langage de base (TypeScript est un système de types pour JavaScript, par exemple).

Pour référence future : on dit d'un langage qui possède une forme de type `Any` qu'il a un *typage graduel* (*gradual typing*).

Ces langages sont dotés d'un grand *écosystème* de bibliothèques *non typées* :

* soit elles ont été écrites avant qu'ils ne se dotent d'un système de types ;
* soit elles préfèrent toujours utiliser le langage de base sous-jacent.

Il est normalement possible de fournir des descriptions typées *externes* des APIs.
Par exemple, c'est le cas de Numpy, ou de [Requests](/series/08-patmat-et-bibliotheques.html#faire-la-requête-depuis-python), ou probablement de la bibliothèque que vous avez choisie pour YAML.
Quand des types externes existent, on peut les installer et en profiter.

Parfois, des bibliothèques n'ont pas de types externes disponibles.
Il reste alors deux solutions :

* les écrire vous-mêmes (ce qui est pénible) ;
* ou manipuler la bibliothèque sur son terrain de jeu, de manière dynamiquement typée, à grands renforts de `Any`.

C'est généralement cette dernière situation qui va donner lieu à un usage responsable de `Any`.
On tentera alors de limiter sa zone d'influence, notamment en isolant les interactions avec cette bibliothèques derrière une bonne encapsulation.

## Slots

Étant donnée sa nature dynamiquement typée, Python a une conception très libre de ce qui fait partie d'un objet.
En particulier avec un objet de type `Any`, on peut même créer des attributs qui n'ont jamais été déclarés ou prévus par sa classe.

```python
class A:
    x: int

    def __init__(self, x: int):
        self.x = x

def main() -> None:
    a: Any = A(5)
    print(a.x)
    a.y = 6 # why not?
    print(a.y) # prints 6
```

Au début du semestre, nous avons introduit les classes comme des sortes de `struct` de C++, avec des méthodes en plus.
À cet éclairage, il est difficile de s'imaginer que l'on puisse ajouter des attributs à une instance.
Effectivement, en C++, cela n'est absolument pas possible.
La définition d'une `struct` prescrit une disposition figée en mémoire.

En Python, c'est différent.
Les attributs d'une instance sont en fait cachés derrière un `dict` magique.
Ce dictionnaire est accessible *via* l'attribut spécial `__dict__`.
Par exemple :

```python
a: Any = A(5)
print(a.x)              # 5
print(a.__dict__)       # {'x': 5}
print(a.__dict__['x'])  # 5
a.y = 6
print(a.y)              # 6
print(a.__dict__)       # {'x': 5, 'y': 6}
print(a.__dict__['y'])  # 6
a.__dict__['z'] = "wow"
print(a.z)              # wow
```

Les noms des attributs sont donc bien des chaînes de caractères à l'exécution.
L'accès à ces attributs se fait, implicitement, en passant par le dictionnaire magique `__dict__`, qui est de type `dict[str, Any]`.
Puisqu'on peut ajouter et retirer des paires clef-valeur dans un `dict`, on peut ajouter et retirer des attributs d'une instance.

C'est évidemment peu recommandé dans le développement de tous les jours.
Dans certaines situations, cela peut être intéressant.

Cette représentation a évidemment un coût.
Plus que de stocker uniquement la valeur des attributs déclarés dans la classe, l'instance doit manipuler un dictionnaire complet, avec des noms de clefs sous forme de `str`.
C'est un coût surtout en terme d'espace mémoire consommé, mais aussi en temps d'exécution.
C'est un des facteurs qui contribuent à la lenteur d'escargot de Python.

Pour la plupart des classes d'un programme, cela n'aura que peu d'importance.
Pour certaines classes critiques, qui auront de très nombreuses instances dans un programme, le coût en mémoire (et en temps d'exécution) peut être trop important.

On peut forcer Python à utiliser une structure plate (sans dictionnaire) pour une classe.
Pour ce faire, on renseigne [l'attribut de classe](./classes-le-retour.html#attributs-de-classe) spécial `__slots__`.
Cet attribut est (normalement) un tuple des noms des attributs de la classe.

```python
class A:
    x: int

    __slots__ = ('x',)

    def __init__(self, x: int):
        self.x = x
```

Lorsqu'une classe possède un attribut de classe `__slots__`, ses instances ne possèderont pas du tout de `__dict__` interne.
À la place, elles prévoieront exactement la place nécessaire pour stocker les attributs spécifiés par `__slots__`, et aucun autre.
Il n'est alors pas possible d'ajouter de nouveaux attributs, ni même d'accéder à `__dict__` :

```python
a: Any = A(5)
print(a.x) # OK

# AttributeError: 'A' object has no attribute '__dict__'.
print(a.__dict__)

# AttributeError: 'A' object has no attribute 'y' and
# no __dict__ for setting new attributes
a.y = 6
```

### Slots et héritage

Lorsqu'une classe a des `__slots__`, il faut que *toutes* ses superclasses, transitives jusqu'à `object`, définissent leurs `__slots__` également.
Si ce n'est pas le cas, les instances auront quand même un `__dict__` en plus des slots déclarés.
Les attributs qui ont été déclarés dans les slots utiliseront les slots, donc tout n'est pas perdu.

```python
class A:
    x: int

    __slots__ = ('x',)

    def __init__(self, x: int):
        self.x = x

class B:
    pass

class C(A, B):
    __slots__ = ('z',)

def main() -> None:
    c: Any = C(5)
    print(c.x)
    print(c.__dict__) # {}
    c.y = 5
    print(c.__dict__) # {'y': 5}
```

Dans le cas d'héritage multiple, seule *une* ligne directe depuis la classe jusqu'à `object` peut définir des `__slots__` *non-vides*.
Si deux parents définissent des `__slots__` non-vides, une exception de type `TypeError` est levée au moment de la définition de la classe.

```python
class A:
    x: int

    __slots__ = ('x',)

    def __init__(self, x: int):
        self.x = x

class B:
    __slots__ = ('y',)

class C(A, B):
    __slots__ = ('z',)
# TypeError: multiple bases have instance lay-out conflict
```

La solution consiste à donner un tuple vide dans `B.__slots__` :

```python
class A:
    x: int

    __slots__ = ('x',)

    def __init__(self, x: int):
        self.x = x

class B:
    __slots__ = ()

class C(A, B):
    __slots__ = ('z',)
```

Notez que, si vous suivez [les recommandations sur les superclasses "de droite"](./classes-le-retour.html#le-cas-de-__init__), en pratique vous pourrez effectivement toujours utiliser un tuple vide dans les superclasses "de droite".
En effet, puisqu'elles n'ont pas d'`__init__` propre, elles n'ont normalement pas non plus d'attribut propre.

Enfin, il ne faut *pas* répéter les noms d'attributs définis dans les superclasses.
Seuls les nouveaux attributs introduits par une classe doivent être mis dans les `__slots__`.

### Exemple de slots

Un exemple que vous avez manipulé sans le savoir est `arcade.Sprite`.
Celle-ci définit en effet des `__slots__` pour tous ses attributs, et sa classe parent `arcade.BasicSprite` fait de même.
Toutefois, j'ai découvert [qu'elle possède quand même un `__dict__`](https://github.com/pythonarcade/arcade/issues/2685) parce qu'elle hérite d'une classe "de droite" qui ne définit pas ses `__slots__`.

Il vaut tout de même mieux suivre le mouvement.
Si vous avez vos propres sous-classes de `arcade.Sprite`, vous devriez définir des `__slots__` appropriés pour réduire la consommation mémoire de votre application.

Bien sûr, à l'échelle de votre projet, cela ne se verra probablement pas.
Arcade le fait parce qu'elle est prévue pour supporter des dizaines et centaines de milliers de sprites.
Vos [tests de performance](/projet/07-extensions.html) pourraient voir la différence, cependant (ou pas).

## SIMD

SIMD, pour **Single Instruction Multiple Data**, sont un ensemble d'instructions sur les processeurs qui manipulent plusieurs données à la fois.
Imaginez que vous avez deux tableaux de 4 `int`s, et que vous voulez calculer leurs sommes deux-à-deux.
Vous devez normalement effectuer 4 instructions d'addition (soit explicitement, soit dans une boucle).

```python
xs = [3, 5, 7, 9]
ys = [8, 7, 13, 4]

zs = []
for i in range(xs):
    zs.append(xs[i] + ys[i])
```

Les instructions de type SIMD permettent d'effectuer *la même instruction* sur des vecteurs parallèles de *données*.
En SIMD, on peut effectuer ces 4 additions en parallèle, de manière efficace, avec une seule instruction machine.

Python ne vous permet pas d'accéder directement aux capacités SIMD de vos processeurs.
C'est une des nombreuses raisons qui font que [NumPy](https://numpy.org/) est un must si vous devez traiter beaucoup de valeurs numériques.
NumPy est écrite en C, et exploite ces instructions dans ses fonctions de base.

Il n'y a rien que vous puissiez faire ici, à part utiliser NumPy quand c'est pertinent.
Sachez simplement que ça existe.

## Manipulation de bits

Finalement, un petit détour au monde fabuleux des manipulations de bits.
Bien sûr, vous vous rappelez que les ordinateurs représentent toutes les données sous forme de séries de bits.
En particulier, les entiers sont exprimés en base 2.
La plupart du temps, on ne s'en rend pas vraiment compte, car on utilise les opérations arithmétiques telles que `+`, `*`, etc.

Cependant, les processeurs ont des instructions très efficaces qui manipulent directement les bits des entiers.
Par exemple, l'opérateur `a & b` donne un résultat `c` tel que chaque bit de `c` est 1 si et seulement les bits correspondants valent 1 dans `a` *et* `b`.
Si `a` et `b` sont des entiers sur 64 bits, cela représente 64 opérations qui sont faites en parallèle !
Et pour un coût du même ordre qu'une simple addition.

Quand on manipule les entiers bit-à-bit, il est souvent utile de les *écrire* directement en base 2.
Peu de langages le permettent, mais en Python c'est possible, avec le préfixe `0b`.
Par exemple, `0b0110 == 6`.

Les opérateurs bit-à-bit (*bitwise operators*) disponibles en Python (et dans presque tous les langages) sont les suivants :

| Opérateur | Signification                | Exemple                      |
|----------:|------------------------------|------------------------------|
|   `a & b` | "et" binaire                 | `0b0011 & 0b0101 == 0b0001`  |
|   `a | b` | "ou" binaire                 | `0b0011 | 0b0101 == 0b0111`  |
|   `a ^ b` | "xor" binaire                | `0b0011 ^ 0b0101 == 0b0110`  |
|      `~a` | "non" binaire, vaut `-a - 1` | `~0b0011 == -0b0100`         |
|  `a << b` | shift left                   | `0b01101 << 3 == 0b01101000` |
|  `a >> b` | shift right                  | `0b01101011 >> 3 == 0b01101` |

Le "non" binaire (*not*) est un peu bizarre en Python, car ses entiers ont tous une précision arbitraire (ils ne sont pas limités à 32 ou 64 bits).
Quand on a un nombre fixe de bits, sa signification est simplement de renverser tous les bits : les 0 deviennent des 1, et inversement.
Avec une précision arbitraire, ça n'a pas beaucoup de sens, car l'infinité implicite de 0s devant le nombre devrait devenir une infinité de 1s.
Avec une interprétation en complément à 2, rappelez-vous qu'on encodait `-x` en renversant tous les bits de `x`, puis en ajoutant 1.
Autrement dit, `-x == ~x + 1`.
Avec un nombre illimité de bits, on définit `~x` dans l'autre sens : `~x == -x - 1`, par définition.

Les shifts correspondent aux multiplications et divisions par des puissances de 2.
`a << b == a * 2**b`, mais en plus efficace.
C'est comme avec la base 10 : si je rajoute 3 zéros à droite d'un autre, je le multiplie par $10^3 = 1000$.
Si je laisse tomber 3 chiffres à droites, je calcule $\lfloor x/10^3\rfloor$.
Il en va de même ici, mais avec des puissances de 2, puisqu'on fonctionne en binaire.

### Exemple de manipulations de bits

Une application typique des manipulations de bits est la représentation efficace d'un *ensemble* de petits entiers.
Si vous devez représenter un ensemble d'entiers compris dans l'intervalle $[0, 40]$, par exemple, vous pouvez le faire avec 41 bits.
L'élément $i$ appartient à l'ensemble représenté par $x$ si le bit $i$ de $x$ vaut 1.
Avec cette représentation, on peut faire les calculs efficaces suivants :

|                                            Mathématiques | Opérations binaires   |
|---------------------------------------------------------:|-----------------------|
|                                               $x \cup y$ | `x | y`               |
|                                               $x \cap y$ | `x & y`               |
|                                          $x \setminus y$ | `x & ~y`              |
|                                                $\\{i\\}$ | `1 << i`              |
|                                   donc, $x \cup \\{i\\}$ | `x | (1 << i)`        |
| test $i \in x$, équiv. $(x \cap \\{i\\}) \neq \emptyset$ | `(x & (1 << i)) != 0` |

Vous trouverez souvent ce type d'encodage pour représenter des "flags" : des ensembles d'options booléennes, avec un nombre fini et petit d'options possibles.
Il est donc bon de connaître cette technique afin de ne pas être totalement perdu-e quand vous tomberez dessus.

Attention : avec des nombres entiers arbitrairement grands, comme en Python, on peut être tenté d'utiliser cette technique même pour de grandes valeurs de $i$.
C'est une mauvaise idée, car la taille de l'ensemble $x$ est alors $\Theta(\text{max}_{i \in x}\; i)$ et non $\Theta(|x|)$.
Si vous utilisez de grands $i$ (> 63), préférez un vrai `set[int]`.
