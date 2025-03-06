---
title: Méthodes spéciales
layout: default
use_mathjax: true
---

# Méthodes spéciales

La semaine dernière, nous avons étudié les concepts de *classe*, d'*attribut* et de *méthode*.
Nous avons vu que les méthodes permettent de créer une *interface* pour une classe, tout en cachant ses détails d'implémentation.

Parmi les méthodes que nous avons définies dans nos classes, `__init__` est clairement spéciale.
C'est une méthode "magique" appelée par Python lors de l'initialisation d'une nouvelle instance.

Il se trouve qu'il existe un grand nombre de ces méthodes dites "spéciales".
Entre autres, elles permettent de définir :

* comment les instances d'une classes doivent être converties en chaînes de caractères pour l'affichage ;
* comment elles répondent aux opérateurs binaires, tels que `+` ;
* comment on les *compare*, notamment avec `==`.

Nous allons étudier aujourd'hui comment se servir des plus fréquentes de ces méthodes spéciales.
Nous reviendrons sur des méthodes spéciales plus avancées vers la fin du semestre.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## `__init__` : constructeur

Nous ne faisons que mentionner `__init__` pour qu'elle apparaisse dans la table des matière.
Nous avons déjà vu en long et en large son fonctionnement dans [le tutoriel sur les classes](./classes.html#attributs-et-constructeur).

## Conversions en chaînes de caractères

### `__repr__` : représentation chaîne developer-friendly

En dehors de `__init__`, la méthode spéciale `__repr__` est sans aucun doute la plus importante.
Elle renvoie une chaîne de caractères supposée représenter l'instance précisément, de manière "developer-friendly".
Sans celle-ci, les instances sont affichées (par exemple dans les messages d'erreur de `pytest`) de manière assez obscure.

Prenons par exemple une classe représentant un vecteur dans le plan.

```python
class Vec:
    x: float
    y: float

    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y
```

Écrivons un test simple qui va échouer :

```python
from vec import Vec2

def test_construction() -> None:
    v = Vec2(5, 7)
    assert v.y == 6
```

Le message d'erreur que nous recevons est bien, mais pas idéal :

```
    def test_construction() -> None:
        v = Vec2(5, 7)
>       assert v.y == 6
E       assert 7 == 6
E        +  where 7 = <vec.Vec2 object at 0x0000021CEAF61940>.y
```

On peut y voir la *représentation* par défaut de notre `Vec2` : `<vec.Vec2 object at 0x0000021CEAF61940>`.
Celle-ci ne nous apprend pas grand chose.
On espérerait voir quelque chose comme `Vec2(5, 7)` à la place.

Nous pouvons obtenir ce résultat en définissant la méthode spéciale `__repr__` comme suit.

```python
class Vec2:
    ... # comme plus haut

    def __repr__(self) -> str:
        return f"Vec2({repr(self.x)}, {repr(self.y)})"
```

Relançons le test, et nous obtenons :

```
    def test_construction() -> None:
        v = Vec2(5, 7)
>       assert v.y == 6
E       assert 7 == 6
E        +  where 7 = Vec2(5, 7).y
```

Ce qui *nettement* plus facile a déboguer.

Notez qu'on utilise `repr(self.x)` pour obtenir la *représentation* de `self.x`.
Dans le cas d'un `float`, ça ne fait aucune différence.
Mais pour des attributs plus complexes qui ont eux-mêmes des méthodes `__repr__`, cela a une importance.
Il est recommandé de toujours utiliser `repr()` pour les attributs au sein de `__repr__`.

`__repr__` ne doit avoir aucun paramètre à part `self`, et renvoyer `str`.
La chaîne renvoyée est supposée être "developer-friendly".
Elle devrait représenter l'instance de la manière la plus précise possible.
Idéalement, cette chaîne est sous forme de code Python (plus ou moins) valide qui permet de recréer la même valeur.
C'est bien le cas pour notre implémentation de `Vec2`.

Dans vos sessions de "débogage", si vous insérez des `print(x)` pour voir les valeurs de `x`, il est parfois mieux d'utiliser `print(repr(x))` à la place lorsque `x` est une instance d'une classe complexe.

`__repr__` est encore utilisée par les outils de développements :

* le REPL de Python l'utilise dans ses résultats ;
* le débogueur (*debugger*, *dévermineur*) dans VS Code l'utilise pour afficher le contenu des variables.

Nous n'avons pas encore parlé du REPL ni du débogueur.
Nous verrons comment nous servir de ces outils lors du troisième cours.

**Cette méthode spéciale aidant énormément pour le débogage, toute classe devrait en être dotée, pour peu que ce soit possible.**

### `__str__` : représentation chaîne user-friendly

La méthode spéciale `__str__` est très similaire à `__repr__`.
Là où le résultat de `__repr__` est destiné aux développeuses et développeurs, celui de `__str__` est destiné aux *utilisateurs*.
Elle est par exemple utilisée par `print(x)`, ainsi que dans les chaînes formattées de la forme `f"... {x} ..."` (en l'absence de `__format__`, voir ci-dessous).

Les utilisateurs et utilisatrices d'un programme de mathématiques n'ont sans doute pas envie de voir les vecteurs affichés sous la forme `Vec2(5, 7)`.
Elles préféreraient voir seulement `(5, 7)`.
C'est moins précis pour le débogage, mais plus user-friendly.

```python
class Vec2:
    ... # comme plus haut

    def __str__(self) -> str:
        return f"({self.x}, {self.y})"
```

La méthode spéciale `__str__` est utilisée par Python dans les circonstances suivantes :

* La fonction standard `str(x)` retourne le résultat de `x.__str__()` ;
* La fonction `print(x)` affiche en réalité `str(x)` ;
* Les chaînes formattées comme `f"La vitesse est le vecteur {x}."` utilisent `__str__` également (enfin presque... voir `__format__` ci-dessous).

En l'absence d'une définition de `__str__`, toutes ces situations utilisent `__repr__` comme représentation par défaut.
Il n'est donc pas utile de définir `__repr__` *et* `__str__` si elles font la même chose.

### `__format__` : représentation chaîne avec options de format

Nous mentionnons cette méthode spéciale plus avancée à titre indicatif.

C'est en fait celle-ci, et non `__str__`, qui est vraiment utilisée par les chaînes de format `f"..."`.
Cependant, en son absence, `__str__` fait office.

Si vous voulez personnaliser la façon dont les instances de votre classe sont [représentées avec des spécificateurs de format](/references/python-vs-cpp.html#formattage-de-chaînes), vous pouvez définir une méthode `__format__`.
Reportez-vous à [la documentation officielle de `__format__`](https://docs.python.org/3/reference/datamodel.html#object.__format__) si cela vous intéresse.

Par exemple, on pourrait vouloir afficher des vecteurs avec une *précision* spécifiée.
Dans ce cas, on voudrait transmettre la précision demandée à la conversion des composantes :

```python
class Vec2:
    ... # comme plus haut

    def __format__(self, format_spec: str) -> str:
        formatted_x = format(float(self.x), format_spec)
        formatted_y = format(float(self.y), format_spec)
        return f"({formatted_x}, {formatted_y})"
```

On peut maintenant contrôler la précision utilisée dans l'affichage de nos vecteurs :

```python
v = Vec2(math.pi, 4.321)
print(f"v = {v}")    # (3.141592653589793, 4.321)
print(f"v = {v:.3}") # (3.14, 4.32)
```

## Égalité vs identité

Pour les classes qui représentent des données immuables, comme notre `Vec2`, on s'attend à ce que `x == y` soit `True` si les *composantes* sont égales.
Pourtant, ce qui n'est pas ce qui se passe :

```python
x = Vec2(5, 7)
y = x
z = Vec2(5, 7)
print(x == y) # True
print(x == z) # False
```

Par défaut, `==` teste si les deux opérandes ont la même *identité*.
C'est le cas de `x` et `y`, puisqu'ils pointent vers la *même instance*.
`z` pointe vers une *autre instance*, bien qu'elle ait le *même contenu*.

![Deux instances de Vec2](/assets/img/vec-eq-vs-is.svg)

Nous voudrions que `x` et `z` soient considérées *égales*, malgré qu'elles n'ont pas la même *identité*.
On peut obtenir ce résultat en définissant la méthode spéciale `__eq__`.
On sera tenté de la définir comme ceci :

```python
class Vec2:
    ... # comme plus haut

    def __eq__(self, other: Vec2) -> bool:
        return self.x == other.x and self.y == other.y
```

Cependant, mypy nous dit

> Argument 1 of `__eq__` is incompatible with supertype `object`; supertype defines the argument type as `object`

Ignorons le problème un instant et exécutons quand même notre petit code d'expérimentation :

```python
x = Vec2(5, 7)
y = x
z = Vec2(5, 7)
print(x == y) # True
print(x == z) # True
```

Tout va bien !

Alors pourquoi `mypy` n'est-il pas content ?
Une vraie réponse à cette question devra attendre encore quelques semaines.
Nous n'avons pas encore tous les concepts pour l'expliquer.

La réponse simplifiée pour l'instant est que `__eq__` est censée accepter *n'importe quel type* de valeur dans `other`.
C'est ce qui est représenté par le type `object`.
Si `Vec2.__eq__` ne reconnaît pas `other` comme une valeur qu'elle comprend, elle doit renvoyer la valeur spéciale `NotImplemented`.

Il faut donc pouvoir déterminer si `other` est un `Vec2`.
Pour l'instant, nous n'avons pas tous les outils pour comprendre comment c'est possible.
Vous allez devoir me croire sur parole que la bonne façon d'implémenter cette méthode est la suivante.

```python
from types import NotImplementedType

class Vec2:
    ... # comme avant

    def __eq__(self, other: object) -> bool | NotImplementedType:
        match other:
            case Vec2():
                return self.x == other.x and self.y == other.y
            case _:
                return NotImplemented
```

Considérez tout ce qui n'est pas la ligne `return self...` comme du "boilerplate".

Notez que `__eq__` redéfinit *l'égalité* pour les instances de `Vec2`.
Il est toujours possible d'accéder à *l'identité* des instances avec l'opérateur `is` :

```python
x = Vec2(5, 7)
y = x
z = Vec2(5, 7)
print(x == y) # True
print(x == z) # True
print(x is y) # True
print(x is z) # False
```

Il est rare de devoir utiliser `is` (dont l'opposé est `is not`).
La seule exception commune est pour tester si quelque chose est `None`, avec `x is None` ou `x is not None`.

Remarque : il est aussi possible de définir `__ne__` qui correspond à l'opérateur `!=`.
Cependant, en l'absence de `__ne__`, `__eq__` est utilisée et on prend son opposé.
Il est donc rare d'avoir réellement besoin de définir `__ne__`.

## Inégalités

Pour les types de données qui admettent un ordre total, vous pouvez aussi définir le comportement des quatre opérateurs de comparaison avec des méthodes spéciales.

| Opérateur | Méthode spéciale | Mnémonique                    |
|-----------|------------------|-------------------------------|
| `<`       | `__lt__`         | **l**ess **t**han             |
| `<=`      | `__le__`         | **l**ess than or **e**qual    |
| `>`       | `__gt__`         | **g**reater **t**han          |
| `>=`      | `__ge__`         | **g**reater than or **e**qual |

Lorsqu'on effectue une comparaison comme `x < y`, cela délègue à un appel de méthode de la forme `x.__lt__(y)`.

Contrairement à `__eq__`, les inégalités n'ont pas à gérer les `object`.
Elles sont plus faciles à définir.

Les vecteurs n'ont pas de relation d'ordre total naturelle.
Voici une implémentation de `__lt__` complètement fausse, pour les besoins de l'illustration.

```python
class Vec2:
    ... # comme avant

    def __lt__(self, other: Vec2) -> bool:
        return self.x < other.x
```

Si vous définissez l'une des quatre inégalités, vous devriez toutes les définir.
Vous devriez aussi définir `__eq__`.

## Opérateurs arithmétiques

De manière similaire aux opérateurs d'inégalités, on peut définir le comportement des opérateurs arithmétiques pour les instances de nos classes.
Les méthodes spéciales pour les opérateurs les plus communs sont les suivantes.

| Opérateur  | Méthode spéciale |
|------------|------------------|
| `+`        | `__add__`        |
| `-`        | `__sub__`        |
| `*`        | `__mul__`        |
| `/`        | `__truediv__`    |
| `//`       | `__floordiv__`   |
| `%`        | `__mod__`        |
| `**`       | `__pow__`        |
| `-` unaire | `__neg__`        |
| `+` unaire | `__pos__`        |

Il peut être intéressant de définir l'addition et la soustraction pour nos `Vec2`, ainsi que les `+` et `-` unaires.

```python
class Vec2:
    ... # comme avant

    def __add__(self, other: Vec2) -> Vec2:
        return Vec2(self.x + other.x, self.y + other.y)

    def __sub__(self, other: Vec2) -> Vec2:
        return Vec2(self.x - other.x, self.y - other.y)

    def __neg__(self) -> Vec2:
        return Vec2(-self.x, -self.y)

    def __pos__(self) -> Vec2:
        return self
```

Ceci nous permet de faire des opérations sur des vecteurs de manière beaucoup plus naturelle :

```python
x = Vec2(5, 7)
y = Vec2(3, 6)
z = -x + Vec2(10, 2) - y
print(z) # (2, -11)
```

### Opérateurs binaires hétérogènes

Les vecteurs s'additionnent et se soustraient entre eux.
Mais ce n'est pas le cas pour la multiplication.
La multiplication sur les vecteurs est hétérogène : c'est un multipliant un vecteur et un scalaire qu'on récupère un autre vecteurs.

On peut commencer par définir `__mul__` dans `Vec2` comme suit.

```python
class Vec2:
    ... # comme avant

    def __mul__(self, other: float) -> Vec2:
        return Vec2(self.x * other, self.y * other)
```

Cela nous permet d'effectuer une opération de type `vec * r`, qui correspond à appeler `vec.__mul__(r)` :

```python
print(Vec2(4, 7) * 2) # (8, 14)
```

En revanche, on ne peut pas écrire `2 * Vec2(4, 7)`.
En effet, cela correspondrait à appeler `2.__mul__(Vec2(4, 7))`.
La méthode `int.__add__` ne sait pas comment réagir face à un `Vec2`.

Pour ces cas de figure, on peut utiliser les méthodes spéciales arithmétiques *renversées* (*reversed*).
Elles portent le même nom que leurs correspondantes, avec un `r` devant.
Par exemple, la version renversée de `__mul__` est `__rmul__`.

Les méthodes renversées sont appelées *sur l'opérande droit*, avec l'opérande gauche en argument.
Python tente une méthode renversée quand la méthode directe n'a pas pu traiter l'opération.

Avec `__rmul__`, nous pouvons ajouter le support pour `2 * Vec(4, 7)`.

```python
class Vec2:
    ... # comme avant

    def __rmul__(self, other: float) -> Vec2:
        return Vec2(other * self.x, other * self.y)
```

```python
print(2 * Vec2(4, 7)) # (8, 14)
```

La présentation ci-dessus est simplifiée par rapport à la réalité.
Elle suffira pour l'instant.
Nous reviendrons plus en profondeur sur ce sujet dans quelques semaines.

## Opérateurs de collections

Avez-vous remarqué que `arcade.SpriteList` supporte les mêmes opérateurs que les `list` de Python ?
On peut obtenir le nombre de sprites dans une `sprite_list` avec `len(sprite_list)`.
On peut aussi accéder à un sprite en particulier avec `sprite_list[index]`.

Ces fonctions primitives et opérateurs de collections, tout comme les opérateurs arithmétiques, ont des méthodes spéciales correspondantes.
Dans le tableau suivant, `T` représente le type des *éléments*.

| Opérateur          | Méthode spéciale              |
|--------------------|-------------------------------|
| `len(x)`           | `x.__len__()`                 |
| `x[i]`             | `x.__getitem__(i)`            |
| `x[i] = new_value` | `x.__setitem__(i, new_value)` |
| `del x[i]`         | `x.__delitem__(i)`            |
| `v in x`           | `x.__contains__(v)`           |

Il est même possible de spécifier comment nos objets réagissent aux boucles `for`.
Ceci est cependant plus avancé.
Nous y reviendrons quand nous aurons plus de concepts à notre disposition.
Pour les curieuses et les curieux, consultez la documentation à propos de la méthode `__iter__`.

## Autres méthodes spéciales

Il y a encore [bien d'autres méthodes spéciales en Python](https://docs.python.org/3/reference/datamodel.html#special-method-names).
Cependant, celles que nous avons vues ici devraient suffire à la plupart des situations que vous rencontrerez.
Les autres sont plus avancées, et sont plus à destination d'expertes et experts Python qui construisent des abstractions pour nous.
