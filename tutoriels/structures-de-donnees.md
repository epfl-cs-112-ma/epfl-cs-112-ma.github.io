---
title: Structures de données
layout: default
use_mathjax: true
---

# Structures de données

Vous avez sans doute remarqué, depuis le temps, qu'il est vraiment *très* commun de manipuler des *listes* dans les programmes.
Ce n'est pas pour rien que Python dispose d'un type standard `list[T]`, que vous avez déjà exploité en long et en large.
Il se trouve qu'il y a trois autres types de "collections" que beaucoup de programmes manipulent : les tuples (`tuple`), les ensembles (`set`) et les dictionnaires (*dictionaries* ou `dict`).
Dans ce chapitre, nous allons explorer ces trois *structures de données*, et ce qu'elles peuvent apporter à nos programmes.

D'autre part, il y a quelques *opérations* sur les collections qui sont tellement fréquentes que Python nous donne de la syntaxe dédiée.
Ce sont les *list/set/dict comprehensions*.
Nous allons aussi découvrir comment elles rendent les manipulations de collections beaucoup plus concises et lisibles.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Listes

### Un type particulier de collection

Une `xs: list[T]` est une collection *ordonnée*, *muable* et *homogène*.

* **Ordonnée** : l'ordre a de l'importance.
  Attention ! Ça ne veut pas dire qu'elle est *triée*, au contraire.
  La liste `[2, 3, 5]` n'est pas la même que la liste `[3, 2, 5]`.
  On peut d'ailleurs demander `xs[1]`, ce qui retourne l'élément à l'index 1 (donc le deuxième élément) de la liste.
  Cela n'a du sens que parce qu'une liste est ordonnée.
  (En mathématiques, un *ensemble* n'est pas ordonné, par exemple : $\\{2,3,5\\} = \\{3,2,5\\}$, et cela n'a donc pas de sens de vouloir prendre le deuxième élément d'un ensemble.)
* **Muable** : il est possible de modifier les éléments d'une liste et de changer sa taille.
  Voir par exemple les opérations `append()`, `remove()`, ou l'assignation `xs[1] = 7`.
* **Homogène** : tous les éléments de `xs` sont d'un même type `T` (ou ses [sous-types](./heritage-et-polymorphisme.html#sous-typage), bien sûr).
  Cela a peu de sens de mettre des `int` et des `str` dans une même liste.
  On *peut* le faire avec une `xs: list[int | str]`, bien sûr, mais ça veut dire que `xs[1]` est de type `int | str` ; et donc il faudrait traiter ces deux cas séparément.
  Sauf cas exceptionnel, ce genre de mélange est le symptôme d'un manque de [polymorphisme](./heritage-et-polymorphisme.html#polymorphisme).

On peut se demander ce que l'on obtient si l'on fait varier ces paramètres.
Par exemple, on peut obtenir un ensemble mathématique (fini) avec les paramètres *non-ordonné*, *immuable* et *homogène*.
En Python, il existe une collection qui possède ces paramètres : un `frozenset`.
Est-ce qu'une collection ordonnée, immuable et *hétérogène* aurait du sens, et serait utile dans les programmes ?
Nous allons voir plus loin que c'est le cas, mais que toutes les combinaisons ne sont pas forcément utiles ou représentées en Python.

### List comprehensions

En mathématiques, vous savez que l'on peut définir un ensemble en *extension* ou en *intension*.
Par exemple :

* extension : $\\{2, 3, 5, 7\\}$, c'est-à-dire l'ensemble des 4 éléments listés explicitement ;
* intension : $\\{x^2 \;\|\; x \in \mathbb{Z} \land 0 \leq x < 10 \land x \equiv 1 \pmod{2}\\}$, c'est-à-dire l'ensemble formés des valeurs de $x^2$ pour tout $x$ entier entre 0 et 10 tels que $x$ est impair.

Vous savez déjà comment définir une liste Python en "extension", avec les `[]` :

```python
# Une liste Python en extension
xs = [2, 3, 5, 7]
```

Pourrait-on définir une liste en "intension" (parfois appelée en "compréhension") ?
Il se trouve que oui !
On peut définir le pendant liste de l'ensemble donné en intension plus haut :

```python
# Une liste en intension, ou "list comprehension"
xs = [x**2 for x in range(0, 10) if x % 2 == 1]
print(xs) # [1, 9, 25, 49, 81]
```

La partie `range(0, 10)` est un peu l'équivalent de $x \in \mathbb{Z} \land 0 \leq x < 10$.
Le reste est une traduction presque "mot à mot" (ou "symbole à symbole") de l'expression mathématique.
Cette liste est donc bien la liste des `x**2` pour chaque `x` dans `range(0, 10)` tel que `x % 2 == 1`.

L'usage de *list comprehensions* est très courant en Python (et dans d'autres langages avec des syntaxes similaires).
Il se trouve qu'il faut souvent "filtrer" les éléments d'une collection (partie `if`) et/ou les "mapper" (leur appliquer une fonction, partie `x**2`) pour créer de nouvelles collections.
Un autre exemple serait d'obtenir le nom des personnes en section maths parmi une liste `students: list[Student]`.
Cela s'écrirait :

```python
students: list[Student] = ...
math_student_names = [s.full_name() for s in students if s.section == Section.MA]
```

Sans les list comprehensions, nous aurions dû écrire toute une boucle pour obtenir le même résultat (c'est ce que vous deviez faire en C++) :

```python
math_student_names = []
for s in students:
    if s.section == Section.MA:
        math_student_names.append(s.full_name())
```

Si on veut les noms de toutes les personnes (c'est-à-dire qu'on mappe mais on ne filtre pas), on omet la section `if` :

```python
all_names = [s.full_name() for s in students]
```

Si on veut les personnes en maths elle-mêmes, on veut filtrer sans mapper.
On utilise alors directement `s` dans la première section :

```python
math_students = [s for s in students if s.section == Section.MA]
```

Une list comprehension peut avoir autant de *clauses* `for` et/ou `if` que nécessaire.
Par exemple, on peut obtenir la liste des chaînes de caractères de *paires* `"(x, y)"` pour tout `x`, `y` dans `range(3)` tels que `x != y` :

```python
xs = [f"({x}, {y})" for x in range(3) for y in range(3) if x != y]
print(xs) # ['(0, 1)', '(0, 2)', '(1, 0)', '(1, 2)', '(2, 0)', '(2, 1)']
```

En pratique, un usage adéquat des list comprehensions permet de se débarrasser de presque toutes les boucles de la plupart des programmes.
Avec un peu d'habitude, cette notation peut être comprise beaucoup plus rapidement par votre cerveau, ce qui vous permet de comprendre les programmes plus aisément.

## Tuples

En introduction, nous avons suggéré qu'il pouvait être utile d'avoir une collection *ordonnée*, *immuable* et *hétérogène*.
Une collection hétérogène a des *types* d'éléments différents en fonction des index.

Si l'on définit la *liste* `xs = [5, "five"]`, son type est au mieux `list[int | str]`.
On a perdu l'information que le premier élément est un `int` et le second une `str`.
On ne peut donc pas écrire `y = x[0] + 1`.

Avec un *tuple*, on garde cette information de manière précise :

```python
t: tuple[int, str] = (5, "five")
y = t[0] + 1 # OK, t[0] est un int
```

Notez que, par construction, nous savons aussi exactement *combien* d'éléments existent dans la collection.

Un tuple se construit comme une liste, mais avec des `()` au lieu de `[]`.
En plus d'être *hétérogènes*, les tuples sont *immuables* : on ne peut pas modifier leurs éléments.

### Utilisation des tuples

Les tuples sont très souvent utilisés de manière "temporaire" lors de calculs plus compliqués.
Par exemple, au hasard, on peut vouloir créer un grand nombre d'objets (peut-être des sprites ?) qui ont presque les mêmes caractéristiques (par exemple une position), mais dont deux dépendent d'un calcul (peut-être une texture et une sprite list dans laquelle le mettre ?).
On peut bien sûr répéter les caractéristiques similaires dans chaque cas :

```python
list_one: list[Thing] = ...
list_two: list[Thing] = ...
x = ...
y = ...
for category in categories:
    match category:
        case Category.ONE:
            thing = Thing(x, y, "one")
            list_one.append(thing)
        case Category.TWO:
            thing = Thing(x, y, "two")
            list_two.append(thing)
```

Cela fonctionne, mais il y a beaucoup de répétitions des choses en commun.
Les répétitions ne sont pas bonnes pour le cerveau, qui peine à identifier les réelles *différences*, et donc les *problèmes* potentiels.
On peut améliorer le code ci-dessus avec un `tuple[str, list[Thing]]` :

```python
for category in categories:
    name_and_list: tuple[str, list[Thing]] # déclaration optionnelle, plus lisible
    match category:
        case Category.ONE: name_and_list = ("one", list_one)
        case Category.TWO: name_and_list = ("two", list_two)

    thing = Thing(x, y, name_and_list[0]) # ok, name_and_list[0] est une str
    name_and_list[1].append(thing)        # ok, name_and_list[1] est une list[Thing]
```

Imaginez s'il y avait 10 catégories ?

### Déconstruction

On peut *déconstruire* un tuple en ses différentes parties en écrivant un "tuple de variables" à gauche de l'assignation :

```python
(name, the_list) = name_and_list
# équivalent à
name = name_and_list[0]
the_list = name_and_list[1]
```

Ce qui donne `name: str` and `the_list: list[Thing]`.
Il est idiomatique en Python d'omettre les parenthèses dans ce cas :

```python
name, the_list = name_and_list
```

ce qui fait la même chose.

Les tuples sont utilisés si une fonction doit renvoyer deux valeurs ou plus.
Puisque Python n'a pas de passage par référence `&` (au sens de C++), on est obligé "d'emballer" plusieurs résultats dans un objet.
Les tuples fournissent une solution légère quand l'emballage n'a pas plus de sens que ça.
Par exemple, la fonction [`divmod`](https://docs.python.org/3/library/functions.html#divmod) de Python renvoie le résultat de la division entière et le modulo en "une opération".
La déconstruction est particulièrement élégante dans ce cas.

```python
div, mod = divmod(10, 3)
print(div) # 3
print(mod) # 1
```

## Tuples comme collections homogènes

Il est aussi possible d'utiliser les tuples comme des collections immuables *homogènes*.
Elles peuvent alors avoir des tailles variables.
Cela se représente par la syntaxe de type suivante, quelque peu alambiquée :

```python
elems: tuple[int, ...] = (2, 3, 5, 7, 11)
```

On les manipule alors comme des `list[int]`, à la différence que les opérations de mutations ne sont pas disponibles.

Il n'y a pas de "tuple comprehensions" en Python.
On ne peut donc pas écrire

```python
squares = (x**2 for x in elems)
```

Enfin : on *peut* écrire cela, mais ça ne donne *pas* un tuple.
Cela donne ce que Python appelle un *générateur*.
Nous reviendrons sur ceux-ci plus tard dans le semestre, lorsque nous nous intéresserons à certains aspects de performances spécifiques à Python.

Il est relativement peu idiomatique d'utiliser les tuples de cette façon en Python.
Même si on n'utilisera une liste que de manière immuable, en Python on a tendance à en faire une `list`, pas un `tuple` homogène.
À mon avis, c'est dommage, mais cela s'explique probablement par l'absence de syntaxe de tuple comprehension.
Les tuples ne sont utilisés comme collection homogène que lorsque la garantie d'immuabilité est vraiment critique.

## Séquences et polymorphisme

Nous avons maintenant vu plusieurs types de données qui représentes des *séquences*, c'est-à-dire des collections ordonnées homogènes (qu'elles soient muables ou pas) :

* `list[T]` : muable, pour tout `T` ;
* `tuple[T, ...]` : immuable, pour tout `T` ;
* mais aussi `str` : immuable, uniquement pour des caractères ;
* `range` : immuable, uniquement pour des `int`.

Beaucoup de fonctions pourraient travailler avec n'importe quelle type de séquence.
Par exemple, on pourrait vouloir écrire une fonction `sum` qui fonctionne avec n'importe quel type de séquence d'entiers.
Elle pourrait fonctionner aussi bien sur des `list[int]`, que des `tuple[int, ...]`, que des `range`.

C'est une occasion en or pour le *polymorphisme*.
Et effectivement, il s'avère que tous ces types sont sous-types de `collections.abc.Sequence[int]`.
On peut donc écrire

```python
from collections.abc import Sequence

def sum(xs: Sequence[int]) -> int:
    result = 0
    for x in xs:
        result += x
    return result

print(sum([1, 2, 3]))        # OK : list[int] <: Sequence[int]
print(sum((2, 3, 5, 7, 11))) # OK : tuple[int, ...] <: Sequence[int]
print(sum(range(10)))        # OK : range <: Sequence[int]

print(sum("hello")) # erreur : str <: Sequence[str] mais pas de Sequence[int]
print(sum(5))       # erreur : int </: Sequence[int]
```

## Sets

Python propose deux types de données pour les *ensembles*, c'est-à-dire les collections *non ordonnées* :

* `set[T]` : non ordonnée, muable, homogène ;
* `frozenset[T]` : non ordonnée, immuable, homogène.

On peut construire un `set[T]` avec une syntaxe similaire aux `list[T]`, mais utilisant `{}` à la place de `[]`.
La syntaxe fonctionne en extension et en intension :

* `{"foo", "bar"}` crée un `set[str]` en extension ;
* `{x**2 for x in range(0, 10) if x % 2 == 1}` crée un `set[int]` en intension.

**Attention :** pour créer un `set` *vide*, il faut utiliser `set()`.
`{}` crée un `dict` vide (voir plus loin).

`frozenset` n'a pas de syntaxe dédiée.
On peut créer un `frozenset` à partir d'un `set` avec `frozenset(s)`.

L'ordre n'a pas d'importance dans les sets.
De plus, ils ne contiennent jamais plusieurs fois la même valeur.
L'ensemble `{2, 4, 2}` crée le même `set` que `{2, 4}` ou `{4, 2}`.

On peut tester l'appartenance à un `set` avec `in` :

```python
xs = {2, 3, 5, 7}
print(5 in xs) # True
print(11 in xs) # False
print(13 not in xs) # True
```

On peut demander la taille d'un set avec `len(xs)`, mais il n'est pas possible *d'indexer* avec `xs[0]`.
En effet, puisque les sets sont non ordonnés, cela n'aurait pas de sens.
On peut cependant itérer sur tous les éléments avec une boucle `for`, ou *via* les list/set comprehensions.

### Opérations sur les sets

Voici d'autres opérations classiques :

| Opération                     | Sens                              |
|-------------------------------|-----------------------------------|
| `x in a`                      | $x \in a$                         |
| `x not in a`                  | $x \notin a$                      |
| `a <= b` ou `a.issubset(b)`   | $a \subseteq b$                   |
| `a < b`                       | $a \subset b$                     |
| `a >= b` ou `a.issuperset(b)` | $a \supseteq b$                   |
| `a > b`                       | $a \supset b$                     |
| `a | b` ("a ou b")            | $a \cup b$                        |
| `a & b` ("a et b")            | $a \cap b$                        |
| `a - b` ("a moins b")         | $a \setminus b$                   |
| `a ^ b` ("a xor b")           | $(a \cup b) \setminus (a \cap b)$ |

Les opération suivantes sont disponibles seulement sur les `set`s muables :

| Opération      | Sens                                                                                 |
|----------------|--------------------------------------------------------------------------------------|
| `a.add(x)`     | ajoute `x` dans `a`                                                                  |
| `a.remove(x)`  | retire `x` de `a` ; `KeyError` s'il n'existe pas                                     |
| `a.discard(x)` | retire `x` de `a` s'il y est ; ne fait rien sinon                                    |
| `a |= b`       | ajoute tous les éléments de `b` à `a`                                                |
| `a &= b`       | retire de `a` les éléments qui ne sont pas dans `b`                                  |
| `a -= b`       | retire de `a` les éléments qui sont dans `b`                                         |
| `a ^= b`       | garde dans `a` les éléments qui sont soit dans `a`, soit dans `b`, mais pas les deux |
| `a.pop()`      | retire un élément "au hasard" de `a`, et le renvoie                                  |
| `a.clear()`    | vide `a`                                                                             |

### Complexité des opérations sur les sets

Il est important de s'interroger sur la *complexité* (au sens de la notation $\Theta$) des opérations sur les ensembles.
En particulier, que coûte un teste d'appartenance `x in a` ?

Si un `set[T]` était implémenté avec une "liste" non triée à l'intérieur, `x in a` aurait besoin d'une recherche linéaire, et serait donc $\Theta(n)$ avec $n$ la taille de l'ensemble (c'est-à-dire `len(a)`).
Si c'était une liste triée, on pourrait une recherche dichotomique et obtenir $\Theta(\log n)$... à condition d'avoir une relation d'ordre total sur les `T` !
Alors à votre avis, quelle est la complexité de `x in a` pour les sets de Python ?

⋮

⋮

⋮

Eh bien non, vous avez perdu !
La complexité est $\Theta(1)$, soit constante.
Il en va de même pour les opérations d'addition (`add`) et de retrait (`remove`, `discard`).

Mais... comment est-ce possible !?
On vous aurait menti en ICC ?
Comment peut-on faire une recherche en temps constant dans un ensemble de taille $n$ ?
Et le plus beau, c'est qu'on n'a même pas besoin de relation d'ordre.

Le catch ?
On a besoin d'une fonction de *hashage* sur les éléments de `T`.
Cette fonction se nomme `hash(x)` en Python, et renvoie un `int`.
Elle doit avoir les propriétés suivantes :

* Si `x == y`, alors il *faut* que `hash(x) == hash(y)`.
* Si `x != y`, alors il *serait bien* que `hash(x) != hash(y)` (c'est-à-dire, cela devrait être vrai avec une grande probabilité).

Évidémment, `==` et `!=` peuvent être personnalisés grâce à la méthode spéciale `__eq__`.
En règle générale, Python ne donc pas savoir comment implémenter `hash` pour tous les types.
Il y a donc, vous l'aurez deviné, une méthode spéciale `__hash__` pour nous permettre de personnaliser nos hash.

### La méthode spéciale `__hash__`

La méthode spéciale `__hash__` n'existe pas pour *tous* les types.
Ce qui veut dire qu'on ne peut pas mettre n'importe quel type d'éléments dans un set.

On dit d'un type $T$ qu'il est *hashable* s'il supporte une `__hash__` adéquate, et peut donc être mis dans un set.
Voici quelques types hashables :

* `int`, `float`, `bool`
* `str`
* `tuple[T1, T2, ..., T3]` si tous les éléments sont eux-mêmes hashables
* `frozenset[T]` si `T` est hashable
* les `@dataclass(frozen=True)`
* vos classes qui n'ont pas de méthode `__eq__` (qui ne devraient alors pas avoir de méthode `__hash__`)
* vos classes qui ont une méthode `__hash__` (correspondant à leur méthode `__eq__`)

Voici quelques types qui ne sont *pas* hashables :

* `list[T]`
* `set[T]`
* les `@dataclass` (avec `frozen=False`)
* vos classes qui ont une méthode `__eq__` mais pas de méthode `__hash__`

On peut voir se dessiner une tendance : sont hashables les types qui

* soit sont immuables ;
* soit ne définissent pas `__eq__`.

Quelle est la propriété partagée par tous ces types ?
C'est que leur *égalité* `==` est *indépendante du temps* :

> Pour toutes instances `x`, `y` de `T`, si `x == y` à un moment de l'exécution du programme, alors `x == y` est *toujours* vrai.

On pourrait exprimer cette propriété en [logique temporelle linéaire](https://fr.wikipedia.org/wiki/Logique_temporelle_linéaire) :

$$ \forall x, y: T. (\textbf{F}(\texttt{x == y}) \rightarrow \textbf{G}(\texttt{x == y})) $$

On peut aisément voir que la réciproque fonctionne aussi : si `x != y` à un moment dans le temps, alors `x != y` est vrai tout le temps.

Cette propriété est critique pour que les algorithmes à l'intérieur de `set`, qui font que `x in a` est en $\Theta(1)$, soient valides.

Pour les classes muables qui ont une `__eq__` personnalisée, on peut vite trouver des contre-exemples :

```python
xs = [1, 2]
ys = [1, 2]
print(xs == ys) # True
xs[1] = 3
print(xs == ys) # False
```

Pour les classes qui ne définissent pas `__eq__`, l'égalité `x == y` est équivalente à *l'identité* `x is y`, qui est bien sûre immuable.

Restent donc les classes immuables qui ont une `__eq__` personnalisée.
Pour ces classes, si l'on veut pouvoir les utiliser dans des `set`s, il faut les rendre hashables en définissant une méthode `__hash__` personnalisée.
Souvent, une égalité personnalisée délègue à l'égalité des composantes.
Par exemple, pour notre bon vieux `Vec2`, nous avions :

```python
class Vec2:
    x: Final[int]
    y: Final[int]

    def __init__(self, x: int, y: int) -> None:
        self.x = x
        self.y = y

    def __eq__(self, other: object) -> bool:
        match other:
            case Vec2():
                return self.x == other.x and self.y == other.y
            case _:
                return NotImplemented
```

Nous voulons donc définir `__hash__` de sorte que :

* si `self.x == other.x and self.y == other.y`,\
  alors `self.__hash__() == other.__hash__()` ;
* sinon, alors `self.__hash__() != other.__hash__()` avec une grande probabilité.

Comment faisons-nous cela ?
On triche !
On réutilise le fait que nous savons que les `tuple`s sont hashables, et on demande donc à Python de calculer un bon hash pour nous :

```python
class Vec2:
    ...

    def __hash__(self) -> int:
        return hash((self.x, self.y))
```

Nous savons que `(self.x, self.y) == (other.x, other.y)` si et seulement si `self.x == other.x and self.y == other.y`.
Nous savons aussi que `hash((self.x, self.y))` nous donnera un "bon" hash (parce que c'est bien fait dans Python).
Donc on en profite !

C'est la méthode recommandée par Python.
N'essayez pas de définir vos propres fonctions de hashage "optimisées".

Avec cette définition, vous pourrez utiliser des `Vec2` dans des `set`s et `frozenset`s.
Remarquez que `Vec2` n'a pas de relation d'ordre.

## Dictionnaires

La dernière structure de données commune est le *dictionnaire* : `dict`.

Un dictionnaire de français associe à chaquet *mot* (une *clef*) sa définition (une *valeur*).
On peut le voir comme un *ensemble de paires* $\\{ (k, v), \ldots \\}$, où chaque $k$ est un mot et chaque $v$ associé sa définition.
Il ne peut pas y avoir deux paires avec la même clef.
D'un point de vue mathématique, cela veut aussi dire que c'est une *fonction* (souvent partielle) des clefs vers les valeurs.

Contrairement à une `list` qui associe des valeurs à des *indices* (entiers consécutifs à partir de 0), les clefs des dictionnaires ont beaucoup plus de liberté.
En Python, un `dict[K, V]` est un dictionnaire muable dont les clefs sont de type `K` et les valeurs sont de type `V`.
`K` doit être un type *hashable*, comme pour les éléments des sets.
Il n'y a pas de contrainte sur `V`.

La recherche d'une clef dans un `dict`, ainsi que l'ajout ou le retrait d'une paire, sont $\Theta(1)$, de manière similaire aux sets.

On peut créer un `dict` avec la syntaxe suivante :

```python
d = {"Police": 117, "Pompiers": 118, "Ambulance": 144, "Pro Juventute": 147}
print(d["Ambulance"]) # 144
```

De manière générale, on écrit les paires clef-valeur avec `clef: valeur`.
`{}` crée un `dict` vide (et non un set).

On peut aussi utiliser une dict comprehension :

```python
d = {x: x**2 for x in range(5)}
print(d) # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

Les principales opérations sur les `dict` sont :

| Opération                  | Sens                                                                                        |
|----------------------------|---------------------------------------------------------------------------------------------|
| `d[key] = value`           | associe `value` à la clef `key` ; remplace une association existante, le cas échéant        |
| `d[key]`                   | retourne la valeur associée à `key` dans `d` ; lance une `KeyError` si la clef n'existe pas |
| `d.get(key, default=None)` | retourne la valeur associée à `key` dans `d`, ou `default` si la clef n'existe pas          |
| `del d[key]`               | retire l'association de la clef `key` ; `KeyError` si la clef n'existe pas                  |
| `key in d`                 | teste s'il y a une association pour `key` dans `d`                                          |
| `key not in d`             | sa négation                                                                                 |
| `list(d)`                  | renvoie une `list` de toutes les clefs dans `d`                                             |
| `items(d)`                 | renvoie une `list` de toutes *paires* `(key, value)` dans `d`                               |
| `len(d)`                   | renvoie le nombre de clefs présentes dans `d`                                               |

Si vous en voulez d'autres, il y en [plus dans la documentation de `dict`](https://docs.python.org/3/library/stdtypes.html#dict).

Toutes les opérations qui travaillent sur une clef `key` donnée sont en $\Theta(1)$.
