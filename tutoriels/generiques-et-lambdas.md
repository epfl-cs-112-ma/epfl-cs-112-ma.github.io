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
    """Reverses a list.

    Returns a new list `ys` such that `len(ys) == len(xs)` and for all index
    `i`, `ys[i] is xs[len(xs) - i - 1]`.
    """
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

### Polymorphisme borné

Ou : un polymorphisme universel pas si universel que ça.
Ou encore : quand on mélange le polymorphisme paramétrique avec le polymorphisme de sous-typage.

Rappelez-vous notre exemple de motivation pour le [polymorphisme par sous-typage](./heritage-et-polymorphisme.html#polymorphisme): la fonction `biggest_rectangle`, qui est devenue plus tard `biggest_shape`.
Nous rappelons les deux versions ici :

```python
def biggest_rectangle(x: Rectangle, y: Rectangle) -> Rectangle:
    if x.area() > y.area():
        return x
    else:
        return y

def biggest_shape(x: Shape, y: Shape) -> Shape:
    if x.area() > y.area():
        return x
    else:
        return y
```

Nous avons présenté `biggest_shape` comme étant *meilleure* que `biggest_rectangle` : on peut l'utiliser dans plus de situations, puisqu'on peut lui donner des `Rectangle`, des `Square`, ou un mix des deux.
Cependant, elle n'est pas *strictement* mieux.
Il y a un "pouvoir" que nous avons perdu :

```python
rect1: Rectangle = ...
rect2: Rectangle = ...
result1: Rectangle = biggest_rectangle(rect1, rect2) # OK
result2: Rectangle = biggest_shape(rect1, rect2) # erreur : renvoie une Shape, pas un Rectangle
```

Si on appelle `biggest_shape` avec deux `Rectangle`, on "sait" qu'on va forcément recevoir un `Rectangle` en retour.
Malheureusement, comme `biggest_shape` prend deux `Shape`, elle doit déclarer renvoyer une `Shape`.

On pourrait tenter de définir `biggest_shape` de manière *générique* pour résoudre ce problème :

```python
def biggest_shape[T](x: T, y: T) -> T:
    if x.area() > y.area(): # erreur : "T" has no attribute "area"
        return x
    else:
        return y
```

Cela permet effectivement à mypy de valider *l'appel* à `biggest_shape`.
Cependant, la *définition* de `biggest_shape` n'est plus valable, puisque `x.area()` nous dit "`T` has no attribute `area`".

Le polymorphisme par sous-typage nous donnait accès à `x.area()`, mais pas le résultat adapté à ce qu'on donne en paramètre.
Le polymorphisme universel nous donne le meilleur type de résultat, mais ne nous donne pas accès à `x.area()`.
Peut-on avoir le beurre et l'argent du beurre ?
Peut-on combiner, d'une manière ou d'une autre, ces deux types de polymorphisme pour avoir accès à `x.area()` *et* obtenir un meilleur type de retour ?

C'est à cette problématique que répond le *polymorphisme borné* (*bounded polymorphism*).
On peut définir `biggest_shape` comme prenant un type de paramètre `T`, mais avec la *contrainte* que `T <: Shape`.
Cela s'écrit avec la syntaxe `[T: Shape]` en Python.

```python
def biggest_shape[T: Shape](x: T, y: T) -> T:
    if x.area() > y.area(): # ok
        return x
    else:
        return y
```

Cette fois, tout passe !
À l'intérieur de `biggest_shape`, nous ne savons pas exactement ce que sera `T`, mais nous savons qu'il sera tel que `T <: Shape`.
Puisque nous pouvons appeler `area()` sur une `Shape`, que `T <: Shape`, et par le Principe de Substitution de Liskov, nous savons que nous pouvons appeler `area()` sur `T`.

Nous avons tous les bénéfices en même temps :

```python
rect1: Rectangle = ...
rect2: Rectangle = ...
square1: Square = ...
square2: Square = ...

result1: Rectangle = biggest_shape(rect1, rect2)  # OK
result2: Square = biggest_shape(square1, square2) # OK
result3: Shape = biggest_shape(rect1, square2)    # OK
result4: Square = biggest_shape(rect1, square2)   # erreur Shape n'est pas un Square
biggest_shape(5, 6) # erreur : int n'est pas un sous-type de Shape
```

Le dernier appel illustre la *constrainte* `T <: Shape`.
On n'a pas le droit d'appeler `biggest_shape` avec `T = int`, puisque `int <: Shape` n'est pas vrai.

### Mieux comprendre votre projet

Vous avez à plusieurs reprises utilisé la fonction `arcade.check_for_collision_with_list`.
Avez-vous regardé sa déclaration ?
Elle utilise une ancienne syntaxe de Python, plus obscure.
Si on la traduit dans la syntaxe moderne, que nous avons étudiée ici, cela donne :

```python
def check_for_collision_with_list[T: BasicSprite](
    sprite: BasicSprite,
    sprite_list: SpriteList[T],
    method: int = 0,
) -> list[T]:
```

Cette fonction exploite le polymorphisme borné.
Vous pouvez lui donner une `SpriteList[T]` pour n'importe quel `T` tel que `T <: BasicSprite` (et souvent, si pas toujours, ce `T` était en fait `Sprite`).
En retour, elle vous garantit qu'elle donne une liste de ces mêmes `T`.
Si vous avez une `class Foo(Sprite)` dans votre programme, et que vous donnez une `SpriteList[Foo]` à `check_for_collision_with_list`, vous avez la garantie de recevoir une `list[Foo]` (et pas juste une `list[Sprite]` ou `list[BasicSprite]`).
Cependant il faut bien que `T <: BasicSprite` soit vrai, sinon elle ne pourrait pas obtenir la hit box des sprites, et donc ne pourrait pas calculer les collisions.

## Lambdas

Avant de s'attaquer aux classes génériques, faisons un petit détour par les *lambdas*.
Les lambdas sont des "fonctions anonymes".
Voyons à quoi cela peut bien servir.

### Une fonction de recherche

Imaginons que nous avons une `list[Person]`, et nous voulons trouver la première personne qui satisfait à un prédicat donné.
Par exemple, qu'elle soit majeure.
Nous pourrions écrire la fonction `find` suivante.

```python
@dataclass
class Person:
    name: str
    age: int

def find_person_of_age(persons: list[Person]) -> Person | None:
    """Finds the first person in the list whose age is `>= 18`.

    If no such person exists, returns `None`.
    """
    for person in persons:
        if person.age >= 18:
            return person
    return None
```

Là aussi, on sent que c'est un motif qui pourrait être récurrent.
Tout comme on voulait des fonctions `reverse` pour n'importe quel *type* de liste, on sent qu'on voudra une fonction `find_person` pour n'importe quel *prédicat*.
D'un point de vue mathématique, vous savez qu'un prédicat est une *fonction* de type $\texttt{Person} \rightarrow \texttt{bool}$.
Pourrait-on écrire une fonction `find_person` qui soit *paramétrée* par une autre fonction `predicate` ?

> Notez que si on voulait *toutes* les personnes satisfaisant ce prédicat (ce qui est plus commun dans les programmes), une list comprehension aurait suffit :
>
> ```python
> [p for p in persons if p.age >= 18]
> ```

### Paramétrer une fonction avec une autre fonction

Il s'avère que Python nous permet de manipuler des fonctions comme n'importe quelle valeur.
La notation pour le *type* d'une fonction $A \rightarrow B$ en Python est `Callable[[A], B]`.
S'il y a plusieurs paramètres (par exemple, $(A_1, A_2, A_3) \rightarrow B$), la notation est `Callable[[A1, A2, A3], B]`.
Le type d'une fonction $\texttt{Person} \rightarrow \texttt{bool}$ s'écrit donc `Callable[[Person], bool]` en Python.

Cela nous permet d'écrire la fonction `find_person` comme suit :

```python
from collections.abc import Callable

def find_person(persons: list[Person], predicate: Callable[[Person], bool]) -> Person | None:
    """Finds the first `person` in the list such that `predicate(person)` is true.

    If no such person exists, returns `None`.
    """
    for person in persons:
        if predicate(person):
            return person
    return None
```

Mais comment *appelle-t-on* cette fonction ?
Il faut lui passer une fonction en argument.
S'il l'on veut reproduire l'exemple des personnes majeures, on aura :

```python
def find_person_of_age(persons: list[Person]) -> Person | None:
    """Finds the first person in the list whose age is `>= 18`.

    If no such person exists, returns `None`.
    """
    def is_of_age(person: Person) -> bool:
        return person.age >= 18
    return find_person(persons, is_of_age)
```

Le type de `is_of_age` est bel et bien `Callable[[Person], bool]`, et est donc un argument valable pour `predicate`.
Lorsqu'on appelle `find_person`, sa boucle appelle `predicate`, qui s'avère être notre fonction `is_of_age`.

Point de terminologie : puisque `find_person` est une fonction paramétrée par une autre fonction, on dit de `find_person` que c'est une *fonction d'ordre supérieur* (*higher-order function*).
C'est la même terminologie qui est utilisée en mathématiques pour des concepts similaires, par exemple la *logique d'ordre supérieur* est une logique dans laquelle on peut paramétrer des prédicats par d'autre prédicats.

**Questionnement :** quel est le type de `find_person` elle-même ?

### Notation plus légère avec les lambdas

D'un point de vue notation, on n'a cependant pas gagné grand chose.
La définition de `is_of_age` prend presqu'autant de caractères, si pas plus, que de récrire la boucle.
Évidemment, ce ne serait pas forcément le cas avec une fonction plus complexe que `find_person`, mais tout de même, ce n'est pas génial.

Pour les situations où nous avons besoin d'une petite fonction (*d'une seule ligne*) uniquement dans le but de la donner en paramètre à une autre fonction, Python nous offre les *lambdas*, ou *fonctions anonymes*.
On peut alors écrire notre fonction `find_person_of_age` vraiment de manière plus concise :

```python
def find_person_of_age(persons: list[Person]) -> Person | None:
    """Finds the first person in the list whose age is `>= 18`.

    If no such person exists, returns `None`.
    """
    return find_person(persons, lambda p: p.age >= 18)
```

En fait, l'implémentation est tellement concise qu'on ne déclarerait même pas `find_person_of_age` !
On appellerait directement `find_person` dans le code qui en a besoin :

```python
of_age_person = find_person(persons, lambda p: p.age >= 18)
```

La syntaxe générale d'une lambda est

```python
lambda param1, ..., paramN: resultat
```

où `resultat` est une expression Python qui peut dépendre des `paramI`.
Il n'y pas de mot-clef `return` ; il est implicite.
Notez qu'on ne donne pas explicitement de types aux paramètres (la syntaxe de Python ne le permet pas).
Python *infère* le bon type en fonction du contexte.
Par exemple, dans l'appel à `find_person`, Python sait qu'il a besoin d'un `Callable[[Person], bool]`, et donc forcément le paramètre `p` doit être de type `Person`.

> **Pourquoi ça s'appelle une lambda ?**
> Historiquement, les fonctions anonymes viennent du [*lambda-calcul*](https://fr.wikipedia.org/wiki/Lambda-calcul) (*lambda calculus*), un modèle mathématique minimal pour la théorie du calcul, développé par Alonzo Church.
> En ICC, nous avons étudié les machines de Turing pour la théorie du calcul.
> Le lambda-calcul est une autre modélisation, *équivalente*, de la théorie du calcul (on peut simuler le lambda-calcul en machine de Turing, et inversement).
> Dans le lambda-calcul original, *toute* valeur est une fonction (anonyme).
> On note une fonction avec la notation $\lambda x. y$, d'où le nom du formalisme.

### Combiner polymorphisme et lambdas

Regardons à nouveau notre fonction `find_person`.

```python
def find_person(persons: list[Person], predicate: Callable[[Person], bool]) -> Person | None:
    """Finds the first `person` in the list such that `predicate(person)` is true.

    If no such person exists, returns `None`.
    """
    for person in persons:
        if predicate(person):
            return person
    return None
```

Remarquez que cette fonction serait tout aussi correcte si on remplaçait `Person` par `Animal` ou `Furniture`.
Elle ne dépend plus vraiment du type des éléments de la liste.
Tout ce dont elle a besoin, c'est que le `predicate` accepte en paramètre les éléments de la liste.

On peut rendre tout ça générique !

```python
def find[T](xs: list[T], predicate: Callable[[T], bool]) -> T | None:
    """Finds the first element `x` in the list such that `predicate(x)` is true.

    If no such element exists, returns `None`.
    """
    for x in xs:
        if predicate(x):
            return x
    return None
```

Nous pouvons maintenant utiliser `find` non seulement avec n'importe quel prédicat, mais aussi avec n'importe quel type de liste.

Il est fréquent en pratique de combiner généricité et fonctions d'ordre supérieur.

## Classes génériques

De la même façon qu'une méthode peut être générique, on peut déclarer des *classes* génériques.
Les classes des collections, `list[T]`, `dict[K, V]`, etc., sont des exemples de classes génériques.

La classe générique la plus simple est probablement une "boîte" pouvant contenir exactement une valeur d'un type `T` donné.
On peut à nouveau commencer par une boîte qui permettrait uniquement de stocker un `int` :

```python
class IntBox:
    value: int

    def __init__(self, value: int) -> None:
        self.value = value

    def swap(self, new_value: int) -> int:
        """Sets the value in the box to `new_value` and returns the previous value."""
        old_value: int = self.value
        self.value = new_value
        return old_value
```

On voit ici que la même implémentation serait parfaitement correcte si on remplaçait tous les `int` par des `str` ou des `Person`s.
C'est donc sans doute une bonne candidate pour être une classe générique :

```python
class Box[T]:
    # à l'intérieur de la classe, on peut utiliser la "variable de type" T
    value: T

    def __init__(self, value: T) -> None:
        self.value = value

    def swap(self, new_value: T) -> T:
        """Sets the value in the box to `new_value` and returns the previous value."""
        old_value: T = self.value
        self.value = new_value # le T de self.value est le même T que celui de new_value
        return old_value
```

Partout à l'intérieur de la classe, on peut utiliser `T`.
C'est le même `T` dans toutes les méthodes, ce qui permet à `swap` de mettre en relation `self.value`, `new_value` et `old_value`

On peut dès lors l'instancier avec n'importe quel type :

```python
int_box: Box[int] = Box(5) # ou explicitement Box[int](5)
int_value: int = int_box.value # ok, value: T mais T = int ici
other_int_value: int = int_box.swap(5) # ok
str_value: str = int_box.value # erreur : int_box.value est un int, pas une str
other_int_value: int = int_box.swap("foo") # erreur

str_box: Box[str] = Box("foo")
str_value: str = str_box.value # ok, T = str ici
other_str_value: str = str_box.swap("bar") # ok
```

Dans notre chapitre sur le [sous-typage](./heritage-et-polymorphisme.html), nous avions dit que chaque *classe* définit un *type* associé.
Ce n'est pas tout à fait le cas pour les classes génériques.
Il existe *une seule* classe `Box`, mais il existe une *infinité* de types associés `Box[U]` (pour tout type `U`).

**Important :** `Box` tout seul (sans `[U]`) n'est *pas* un type valide.
Il n'est pas possible de définir une variable de type `x: Box`.
Si on essaye, on obtient une erreur comme "Missing type parameters for generic type `Box`".
Inversement, `Box[int]` est un type valid mais n'est *pas* une classe.

### Héritage entre classes génériques

Une classe générique peut hériter d'une autre classe générique, ou d'une classe non générique (aussi dite *monomorphique*, par opposition à *polymorphique*).
Inversement, une classe monomorphique peut hériter d'une classe polymorphique.
Dans tous les cas, si la classe parent est générique, il faut lui fournir les arguments de types qui vont bien.

Voilà deux exemples de classes qui pourraient étendre de `Box` :

```python
class PowerBox[U](Box[U]):
    def compare_and_swap(self, expected_old_value: U, new_value: U) -> bool:
        """Sets the value in the box to `new_value` if the old value is the expected one.

        Returns `True` iff the old value was the expected one and the
        modification was performed.
        """
        if self.value == expected_old_value:
            self.value = new_value
            return True
        else:
            return False

class IntBox(Box[int]):
    def inc(self) -> int:
        """Increments the value in the box, then returns the new value."""
        new_value: int = self.value + 1
        self.value = new_value
        return new_value

class PairBox[U](Box[tuple[U, U]]):
    def set_both(self, x: U) -> None:
        """Sets the value in the box to pair where both elements are the same value `x`."""
        self.value = (x, x) # x: U donc (x, x): tuple[U, U]
```

Notez qu'on a explicitement utilisé une autre lettre, `U`, pour `PowerBox`, pour des raisons pédagogiques.
Par la relation d'héritage, au sein de la classe `PowerBox`, nous savons que le `U` de `PowerBox` est le *même* type que le `T` de `Box`.
C'est pourquoi on a le droit de faire `self.value = new_value` : `self.value` est du type `T` de `Box`, et `new_value` est du type `U` de `SuperBox`.
Dans un vrai programme, il est souvent plus clair d'utiliser la même lettre dans ce cas.

Mais ça ne doit pas vous empêcher de comprendre que c'est fondamentalement *une autre variable de type* (qui porte le même nom).
C'est similaire aux paramètres des fonctions :

```python
def foo(x: int) -> int: ...

def bar(x: int) -> int:
    return foo(x) + 1
```

Ces deux `x` sont bien deux variables distinctes, même si `bar(x)` appelle `foo(x)`.

Dans le cas de `IntBox`, la relation d'héritage établit que le `T` de `Box` est `int`.
Et donc au sein de cette classe, on peut faire `self.value + 1`, par exemple.

Dans `PairBox`, nous savons que le `T` de `Box` est en fait `tuple[U, U]` où `U` est celui de `PairBox`.
Ici, il est plus judicieux de réllement utiliser deux noms différents, puisqu'ils ne veulent vraiment pas dire la même chose.

On dérive des relations de sous-typage à partir de ces déclarations d'héritage :

* $\forall S. \texttt{SuperBox[}S\texttt{]} <: \texttt{Box[}S\texttt{]}$
* $\texttt{IntBox} <: \texttt{Box[int]}$
* $\forall S. \texttt{PairBox[}S\texttt{]} <: \texttt{Box[tuple[}S\texttt{, }S\texttt{]]}$
