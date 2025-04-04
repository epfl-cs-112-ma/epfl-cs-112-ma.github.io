---
title: Pattern matching
layout: default
use_mathjax: true
---

# Pattern matching

Vous savez déjà comment utiliser `match` pour des cas simples : tester les cas d'un `Enum`, ou tester si l'argument de `__eq__` est de votre classe, etc.
Dans ce chapitre, nous voyons plus en détail ce que peut faire `match`.
En général, cette instruction est capable de faire du *pattern matching* (reconnaissance de motifs), ce qui lui donne son nom.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Rappels

Pour l'instant, il y a surtout deux façons dont vous utilisez `match`.
La première, c'est pour tester les différentes valeurs possibles d'un `Enum`.

```python
from enum import Enum, auto

class RoadLight(Enum):
    GREEN = auto()
    ORANGE = auto()
    RED = auto()

def ok_to_cross(light: RoadLight) -> bool:
    match light:
        case RoadLight.GREEN | RoadLight.ORANGE:
            return True
        case RoadLight.RED:
            return False
```

La seconde, c'est dans le cas particulier de `__eq__`.
Cette méthode doit prendre un paramètre de type `object` pour respecter le LSP.
À l'intérieur, on veut *tester* si l'argument est du type de notre classe, ce qu'on peut faire avec un `match`.

```python
class Vec2:
    ...
    def __eq__(self, other: object) -> bool:
        match other:
            case Vec2():
                # ici nous savons (et mypy aussi) que `other: Vec2`
                return self.x == other.x and self.y == other.y
            case _:
                return NotImplemented
```

## Exhaustivité et le type `Never`

Reprenons le cas de l'enum `RoadLight` ci-dessus, et introduisons un bug dans `ok_to_cross` : on a oublié de traiter le cas `ORANGE`.

```python
def ok_to_cross(light: RoadLight) -> bool:
    match light:
        case RoadLight.GREEN:
            return True
        case RoadLight.RED:
            return False
```

Mypy ne nous laisse pas faire !
Nous recevons un message d'erreur sur la définition de cette fonction :

> Missing return statement

Il est malheureusement un peu obscur.
Manifestement, nous avons bien mis des instructions `return`.
Cependant, mypy comprend que notre `match` n'est pas *exhaustif*.
Il y a des cas qui ne sont pas traités, et donc pour lesquels il n'y a pas de `return` associé.

Malheureusement, mypy ne vérifie pas toujours l'exhaustivité.
Il le fait pour les `return`, mais pas pour l'initialisation des variables locales.
Il est possible de contourner la vérification de mypy de la manière suivante :

```python
def ok_to_cross(light: RoadLight) -> bool:
    result: bool
    match light:
        case RoadLight.GREEN:
            result = True
        case RoadLight.RED:
            result = False
    return result
```

Oops ! Mypy ne montre pas d'erreur ici.
Évidemment, on n'écrirait pas cette fonction particulière de cette façon ; les `return` directement dans les `case` sont bien plus clairs.
Mais dans des fonctions plus longues, il est fort possible que cela arrive.
D'ailleurs, beaucoup d'entre vous ont un schéma similaire quelque part dans leur projet.

On peut forcer mypy à vérifier l'exhaustivité du `match` en ajoutant un `case` un peu particulier.

```python
from typing import assert_never

def ok_to_cross(light: RoadLight) -> bool:
    result: bool
    match light:
        case RoadLight.GREEN:
            result = True
        case RoadLight.RED:
            result = False
        case _:
            assert_never(light)
    return result
```

Cette fois, on obtient une erreur dans le `assert_never` :

> Argument 1 to `assert_never` has incompatible type `Literal[RoadLight.ORANGE]`; expected `Never`

Alors que si l'on rajoute le cas `ORANGE`, il n'y a plus d'erreur.

Techniquement, `assert_never` vérifie que son argument `light` a bien le type `Never`.
Pourtant `light` est de type `RoadLight`, pas `Never`.

La raison est que mypy est capable de *raffiner* le type de `light` petit à petit.
Au départ, il est de type `RoadLight`.
Mais il sait aussi que `RoadLight` peut être vu comme *l'union* des différents cas de l'énumération, un peu comme si il était défini avec un alias de type.

```python
type RoadLight = RoadLight.GREEN | RoadLight.ORANGE | RoadLight.RED
```

Dans le `case GREEN | ORANGE`, mypy peut raffiner le type de `light` comme l'union `GREEN | ORANGE`.
C'est similaire au fait qu'il raffine `other: Vec2` dans le `case Vec2()` de `__eq__`.

Plus intéressant, si ce cas n'est *pas* sélectionné, mypy sait qu'il lui reste `RoadLight \ (GREEN | ORANGE)`.
Le `\` représente ici la "différence" de type, par analogie à la différence d'ensemble.
Rappelez-vous que les types représentent des ensembles de valeurs.
Puisque `RoadLight` est l'union `GREEN | ORANGE | RED`, après le premier `case`, il ne lui reste plus que `RED`.

Dans le `case RED`, mypy sait de la même manière que `light: RED`, bien que ça ne soit pas très utile.
Mais il sait aussi qu'après ce case, il ne lui reste plus que `RED \ RED`.
Que peut être ce type ?
Sur les ensembles, $S \setminus S = \emptyset$.
Par analogie, le type `RED \ RED` devrait être un type qui représente l'ensemble vide de valeurs.
C'est précisément ce que représente le type spécial `Never`.

Je répète : le type `Never` (`from typing import Never`) représente l'ensemble vide de valeurs.

Par analogie au fait que, pour les ensembles, $\forall S. \emptyset \subseteq S$, nous avons la règle de sous-typage suivante :

$$\forall T. \texttt{Never} <: T$$

Puisque la fonction `assert_never(light)` demande un `Never` en paramètre, et que `light` a été *raffiné* jusqu'à ce devenir `Never`, mypy n'émet pas d'erreur.

Si en revanche le cas `ORANGE` n'est pas traité, le type de `light` n'aurait été raffiné qu'au singleton `ORANGE`, qui n'est *pas* un sous-type de `Never`, causant ainsi le message d'erreur de mypy :

> Argument 1 to `assert_never` has incompatible type `Literal[RoadLight.ORANGE]`; expected `Never`

La vérification automatiquement de l'exhaustivité est un autout majeur du pattern matching.
Si vous utilisez un `match` qui est censé être exhaustif, et qui ne fait pas des `return` dans toutes ses branches, vous devriez *toujours* ajouter le `assert_never` correspondant.

En plus de nous protéger de l'oubli immédiat d'un cas lorsqu'on écrit le `match`, l'exhaustivité nous aide dans le refactoring.
Souvent, un `Enum` va amasser plus de cas avec le temps.
À chaque fois que vous ajoutez un cas, la vérification d'exhaustivité de mypy vous indiquera automatiquement *tous* les endroits où vous devez tenir compte de la nouvelle valeur possible.

⚠️ Ne pas confondre `Never` et `None`.
`None` est le type qui représente le *singleton* $\\{ \texttt{None} \\}$, et admet donc une seule valeur, à savoir `None`.
`Never` est le type qui représente *l'ensemble vide* $\emptyset$.
Il n'admet donc *aucune* valeur, pas même `None`.

En pratique, si un morceau de code a "dans les mains" (à disposition) une variable de type `Never`, c'est que ce morceau de code ne peut jamais être exécuté.
On dit que c'est du *dead code* (code mort).
C'est ce qui donne son nom au type `Never`.
Du code qui a une variable de type `Never` "can never happen".

**Self-check** : est-ce qu'utiliser `assert_never` dans le cas du `match other` de `__eq__` a du sens ?

## Motifs

Lorsqu'on utilise `match` pour tester un plusieurs *types* de données, comme avec dans le cas de `__eq__`, on peut automatiquement extraire les attributs qui nous intéressent.
Reprenons la méthode `Vec2.__eq__` que nous connaissons :

```python
class Vec2:
    ...
    def __eq__(self, other: object) -> bool:
        match other:
            case Vec2():
                # ici nous savons (et mypy aussi) que `other: Vec2`
                return self.x == other.x and self.y == other.y
            case _:
                return NotImplemented
```

Dans le `case Vec2()`, on utilise `other.x` et `other.y`, en s'appuyant sur le type *raffiné* de `other`.
Une autre possibilité est de profiter du *motif* (*pattern*) pour extraire les attributs dans des variables locales :

```python
class Vec2:
    ...
    def __eq__(self, other: object) -> bool:
        match other:
            case Vec2(x=other_x, y=other_y):
                return self.x == other_x and self.y == other_y
            case _:
                return NotImplemented
```

Dans cet exemple particulier, ce n'est pas très utile.
D'aucun diraient que c'est même contreproductif.

Ce qui est intéressant, c'est que l'on peut récursivement placer des *motifs* à la droite des symboles `=`.
Reprenons par exemple [l'exercice sur la dérivation formelle](/series/01-appropriation-de-python.html#dérivation-formelle).
Vous avez probablement fini avec un code ressemblant à ceci dans la fonction `derive` :

```python
def derivative(expr: Tree, x: str) -> Tree:
    """Computes the formal derivative of the given expression."""
    if expr.left is None or expr.right is None:
        if expr.value == x:
            return leaf('1')
        else:
            return leaf('0')
    else:
        f = expr.left
        g = expr.right
        df = derivative(f, x)
        dg = derivative(g, x)
        match expr.value:
            case '+':
                return tree('+', df, dg)
            case '-':
                return tree('-', df, dg)
            case '*':
                return tree('+', tree('*', df, g), tree('*', f, dg))
            case '/':
                return tree('/', tree('-', tree('*', df, g), tree('*', f, dg)), tree('*', g, g))
            case '^':
                # We assume here that g = a independent of x, as per the statement
                a = g
                return tree('*', tree('*', a, df), tree('^', f, tree('-', a, leaf('1'))))
            case _:
                raise ValueError(f"Unknown operator {expr.value}")
```

Nous allons maintenant voir comment l'usage des motifs du `match` permettent de significativement réduire ce code, et le rendre plus lisible.

## Extraction des valeurs

D'abord, récrivons le premier `if` en `match` comme suit :

```python
match expr:
    case Tree(value=v, left=None, right=None):
        if v == x:
            return leaf('1')
        else:
            return leaf('0')
    case _:
        ...
```

Le motif dit ceci :

* si `expr` est un `Tree` (ça c'est forcément vrai à cause du type de `expr`) ;
* et si `expr.left is None` ;
* et si `expr.right is None` ;
* alors le motif *correspond* (*matches*) :
    * extraire la valeur de `expr.value` dans la variable locale `v` ;
    * et entrer dans le `case` ;
* sinon, le motif ne correspond pas :
    * passer au motif suivant.

Notez que n'est pas tout à fait équivalent au premier `if`.
Ici, nous exigeons que `left` et `right` soient *tous les deux* `None`.

**Attention** : notez la subtile différence entre `value=v` et `left=None`.
`value=v` correspond toujours, et assigne à `v` la valeur de l'attribut `value`.
`left=None` ne correspond que si l'attribut `left` vaut `None`.
Seuls les identificateurs spéciaux `None`, `True` et `False` ont ce comportement.
Tout autre identificateur simple (sans `.`) correspond toujours et assigne une variable locale.

Le motif `_` correspond toujours.
C'est un joker.

## Gardes

On peut intégrer le `if v == x` comme *garde* (*guard*) du `case`.
Ceci en fait alors deux `case` différents.

```python
match expr:
    case Tree(value=v, left=None, right=None) if v == x:
        return leaf('1')
    case Tree(left=None, right=None):
        return leaf('0')
    case _:
        ...
```

Ceci ajoute une condition au premier `case`.
Après avoir extrait `v = expr.value`, on teste encore si `v == x` avant de décider que `expr` coorespond au motif.
On n'entre dans le `case` que si c'est le cas.

Si le premier motif ne correspond pas, on passe au suivant.
Puisqu'on a déjà éliminé le cas `v == x`, on n'a plus besoin de tester `value` dans ce deuxième cas.
Du moment qu'on a `left is None` et `right is None`, on est dans le cas où il faut renvoyer `leaf('0')`.

On peut mettre n'importe quelle expression booléenne dans une garde.

## Motifs imbriqués

Après avoir géré les cas de variables ou constante, nous devons gérer les cas d'opérateurs.
Pour les opérateurs, `left` et `right` doivent tous les deux être des sous-arbres (plutôt que `None`).
On peut tester tout cela en une seule fois avec des *motifs imbriqués*.
À droite du `=`, on peut avoir un autre motif.

```python
case Tree(value=op, left=Tree(), right=Tree()):
    ...
```

Ce motif correspondra si `expr.left` et `expr.right` sont eux-mêmes des `Tree`.
Dans ce cas, on extrait l'opérateur stocké dans `expr.value` dans la variable `op`.
Cependant, on va être un peu bloqué, parce qu'on n'a pas extrait les sous-arbres `left` et `right` dans des variables locales.
Si on réaccède à `expr.left`, on reçoit de nouveau un `TreeOpt` (`Tree | None`), ce qui ne nous arrange pas.

On veut donc à la fois tester un sous-motif, *et* extraire la valeur dans une variable locale si ce sous-motif correspond.
On peut le faire avec le mot-clef `as`.

```python
case Tree(value=op, left=Tree() as f, right=Tree() as g):
    # f: Tree et g: Tree ici
    ...
```

## Égalité littérale

Nous en sommes à ceci :

```python
def derivative(expr: Tree, x: str) -> Tree:
    """Computes the formal derivative of the given expression."""
    match expr:
        case Tree(value=v, left=None, right=None) if v == x:
            return leaf('1')
        case Tree(left=None, right=None):
            return leaf('0')
        case Tree(value=op, left=Tree() as f, right=Tree() as g):
            df = derivative(f, x)
            dg = derivative(g, x)
            match op:
                case '+':
                    return tree('+', df, dg)
                case '-':
                    return tree('-', df, dg)
                case '*':
                    return tree('+', tree('*', df, g), tree('*', f, dg))
                case '/':
                    return tree('/', tree('-', tree('*', df, g), tree('*', f, dg)), tree('*', g, g))
                case '^':
                    # We assume here that g = a independent of x, as per the statement
                    a = g
                    return tree('*', tree('*', a, df), tree('^', f, tree('-', a, leaf('1'))))
                case _:
                    raise ValueError(f"Unknown operator {op}")
        case _:
            raise ValueError(f"Invalid tree {expr}")
```

Ce qui est bien, mais ce n'est pas clair qu'on a vraiment gagné quelque chose.
Nous pouvons cependant encore remonter le `match op` comme des cas distincts du grand `match expr`.
En effet, nous pouvons intégrer ces littéraux (les constantes comme `'+'`) comme sous-motifs dans `value=`.

Petit bémol dans ce cas précis, on n'a plus d'endroit valable pour définir `df` et `dg`.
On va utiliser une petite fonction locale à la place, pour ne pas perdre en lisibilité.

```python
def derivative(expr: Tree, x: str) -> Tree:
    """Computes the formal derivative of the given expression."""
    def d(f: Tree) -> Tree:
        return derivative(f, x)

    match expr:
        case Tree(value=v, left=None, right=None) if v == x:
            return leaf('1')
        case Tree(left=None, right=None):
            return leaf('0')

        case Tree(value='+', left=Tree() as f, right=Tree() as g):
            return tree('+', d(f), d(g))

        case Tree(value='-', left=Tree() as f, right=Tree() as g):
            return tree('-', d(f), d(g))

        case Tree(value='*', left=Tree() as f, right=Tree() as g):
            return tree('+', tree('*', d(f), g), tree('*', f, d(g)))

        case Tree(value='/', left=Tree() as f, right=Tree() as g):
            return tree('/', tree('-', tree('*', d(f), g), tree('*', f, d(g))), tree('*', g, g))

        case Tree(value='^', left=Tree() as f, right=Tree() as g):
            # We assume here that g = a independent of x, as per the statement
            a = g
            return tree('*', tree('*', a, d(f)), tree('^', f, tree('-', a, leaf('1'))))

        case _:
            raise ValueError(f"Invalid tree {expr}")
```

Il nous reste une dernière amélioration que nous pouvons faire facilement ici.
Comme `Tree` est une `@dataclass`, on peut omettre les `value=`, `left=` et `right=`.
Du moment que l'on respecte *l'ordre de définition de ces attributs*, le `match` sait comment associer les sous-motifs aux bons attributs.
Cela rend la lecture de notre `match` bien plus aisée, en éliminant le "bruit" parasite à notre compréhension.

```python
def derivative(expr: Tree, x: str) -> Tree:
    """Computes the formal derivative of the given expression."""
    def d(f: Tree) -> Tree:
        return derivative(f, x)

    match expr:
        case Tree(v, None, None) if v == x:
            return leaf('1')
        case Tree(_, None, None):
            return leaf('0')

        case Tree('+', Tree() as f, Tree() as g):
            return tree('+', d(f), d(g))

        case Tree('-', Tree() as f, Tree() as g):
            return tree('-', d(f), d(g))

        case Tree('*', Tree() as f, Tree() as g):
            return tree('+', tree('*', d(f), g), tree('*', f, d(g)))

        case Tree('/', Tree() as f, Tree() as g):
            return tree('/', tree('-', tree('*', d(f), g), tree('*', f, d(g))), tree('*', g, g))

        case Tree('^', Tree() as f, Tree() as g):
            # We assume here that g = a independent of x, as per the statement
            a = g
            return tree('*', tree('*', a, d(f)), tree('^', f, tree('-', a, leaf('1'))))

        case _:
            raise ValueError(f"Invalid tree {expr}")
```

Notez qu'on a dû introduire un sous-motif `_` dans le deuxième `case`, pour correspondre à l'attribut `value`.
Si on ne le fait pas, le motif correspondrait lorsque `value` et `left` sont `None` (à la place de `left` et `right`).

## Aparté : types algébriques de données (ADT)

Il y a encore une amélioration que nous pouvons faire sur cet exemple.
Celle-ci requiert un refactoring plus important.
Pour l'instant, `TreeOpt = Tree | None`, et on représente les feuilles de l'arbre comme des `Tree` qui se trouvent avoir des `None` dans les sous-arbres.
Ce n'est pas vraiment idéal.
À cause de ça, on n'a pas de lien fort entre le fait d'avoir un *opérateur* et le fait d'avoir de réels sous-arbres.

Maintenant que vous connaissez le polymorphisme, il y a une bien meilleure manière d'encoder ce type d'arbre binaire.
On va donner des *types différents* aux feuilles et aux branches.

```python
type Tree = Leaf | Branch
"""Binary tree, with a new representation."""

@dataclass(frozen=True)
class Leaf:
    """Leaf, with only a value."""
    value: str

@dataclass(frozen=True)
class Branch:
    """Binary operator, with two subtrees."""
    operator: str
    left: Tree
    right: Tree
```

On appelle ce type de représentations de données un *type algébrique de données* (*algebraic data type* ou **ADT**).
On leur donne ce nom car elles correspondent directement à des notions algébriques telles que les *sommes* et les *produits cartésiens* d'ensembles.
Si vous voulez plus d'informations, [la page Wikipédia sur les ADTs](https://fr.wikipedia.org/wiki/Type_alg%C3%A9brique_de_donn%C3%A9es) est un bon point de départ.

On va pouvoir profiter du pattern matching de manière bien plus efficace avec les ADTs.
Pour prendre un exemple simple, voyons comment récrire la fonction `tree_to_str` (oui, je sais qu'aujourd'hui vous aurez compris que cette fonction devrait plutôt être la méthode spéciale `__str__`).

```python
def tree_to_str(tree: Tree) -> str:
    match tree:
        case Leaf(value):
            return f"({value})"
        case Branch(op, left, right):
            left_str = tree_to_str(left)
            right_str = tree_to_str(right)
            return f"({left_str} {op} {right_str})"
```

On peut voir ici la forme caractéristique d'une fonction qui travaille sur un ADT.
On a un `match` sur la valeur de ce type, et un `case` par `@dataclass` faisant partie de l'union.
Dans chaque `case`, on extrait les valeurs des attributs, et on effectue un calcul dessus.

Notez que si vous oubliez un des cas, mypy détectera que votre `match` n'est pas exhaustif.

On peut exploiter cette nouvelle structure pour écrire `derivative` de manière encore plus claire.
À ce stade, on atteint ce qui se ferait dans un logiciel professionnel (à la différence qu'on utiliserait encore un `Enum` pour les opérations possibles).

```python
def derivative(expr: Tree, x: str) -> Tree:
    """Computes the formal derivative of the given expression."""
    def d(f: Tree) -> Tree:
        return derivative(f, x)

    match expr:
        case Leaf(v) if v == x:
            return leaf('1')
        case Leaf(_):
            return leaf('0')

        case Branch('+', f, g):
            return tree('+', d(f), d(g))

        case Branch('-', f, g):
            return tree('-', d(f), d(g))

        case Branch('*', f, g):
            return tree('+', tree('*', d(f), g), tree('*', f, d(g)))

        case Branch('/', f, g):
            return tree('/', tree('-', tree('*', d(f), g), tree('*', f, d(g))), tree('*', g, g))

        case Branch('^', f, g):
            # We assume here that g = a independent of x, as per the statement
            a = g
            return tree('*', tree('*', a, d(f)), tree('^', f, tree('-', a, leaf('1'))))

        case Branch(op, _, _):
            raise ValueError(f"Invalid operator {op} of tree {expr}")
```

Le dernier cas ne serait pas nécessaire si `operator` était un `Enum`, puisque nous aurions la garantie d'avoir traité de manière exhaustive tous les cas.

## Motifs de collections

Il y a encore quelques motifs spécifiques aux collections standard de Python.
On peut extraire les éléments de `tuple`s, `list`s et `dict`s (et tester au passage que les valeurs sont bien des types correspondants).
On utilise pour cela la même syntaxe que pour *construire* ces collections, mais cette fois pour les *déconstruire* dans des motifs.

```python
match value:
    case (x, y):
        # tuple avec deux éléments, x et y
        ...
    case (x, *ys):
        # tuple avec 1 valeurs ou plus
        # x est la première, ys est une list des autres
        ...
    case [x, int() as y]:
        # list avec deux éléments, le second étant un int
        ...
    case [x, y, *zs]:
        # list avec aux moins deux éléments
        # les deux premiers sont dans `x` et `y`, les autres dans `zs`
        ...
    case [1, 2, *zs]:
        # même chose mais les deux premiers éléménts doivent être 1 et 2
        ...
    case {"foo": x, "bar": y}:
        # dict qui contient *au moins* les clefs "foo", et "bar"
        # x et y sont les valeurs associées à ces clefs
        ...
```
