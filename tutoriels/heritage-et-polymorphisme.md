---
title: Héritage et polymorphisme
layout: default
use_mathjax: true
---

# Héritage et polymorphisme

Cette semaine, nous abordons ce qui est sans doute le point le plus important de tout notre cours : l'héritage et le sous-typage.
Dans un langage orienté objet, ces deux notions sont liées, bien que distinctes.

L'héritage est la capacité pour une classe de réutiliser (hériter) un certain nombre de comportements déjà implémentés par une classe existante.
L'héritage concerne donc les *valeurs* instances de classes données.

Le sous-typage est une propriété des *types* du langage.
Lorsqu'un type $A$ est un *sous-type* d'un autre type $B$, on peut utiliser une expressione de type $A$ là où on attend un $B$.

Vous avez en fait déjà utilisé l'héritage et le sous-typage, sans nécessairement le savoir.
Dans le projet, votre `GameView` *étend* la classe `arcade.View`, ce qui est une application de l'héritage.
Quant au sous-typage, vous avez déjà manipulé dans les exercices des types de la forme `A | None`.
Vous avez pu assigner autant des `A` que des `None` à des variables de type `A | None`.
C'est parce que `A` et `None` sont des *sous-types* de `A | None`.

Une fois combinées, les notions d'héritage et de sous-typage nous amèneront au *polymorphisme* : la capacité d'une fonction à traiter *uniformément* des valeurs de *types différents*.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Sous-typage

### Intuition

Vous êtes-vous déjà réellement posé la question de ce qu'*est* un type ?
Fondamentalement, un *type* $T$ représente un *ensemble de valeurs*.
Dans ce sens, on dit qu'une valeur $x$ a le type $T$ si $x$ appartient à l'ensemble de valeurs représenté par $T$.

Intuitivement, le sous-typage correspond alors à *l'inclusion* (non stricte) entre ensembles.
On dit du type $S$ qu'il est sous-type de $T$ si toutes les valeurs de type $S$ ont aussi le type $T$.
Autrement dit, si $S$ représente un sous-ensemble de l'ensemble représenté par $T$.

Par exemple, voici quelques "sortes" de types que vous connaissez déjà :

* le type `int` représente l'ensemble de toutes les valeurs entières : $\\{ \ldots, -2, -1, 0, 1, 2, \ldots \\}$ ;
* le type `None` représente le singleton $\\{ \texttt{None} \\}$ (bien que *notés* de la même façon, ces deux `None` sont bien distincts ; on les reconnaît par le contexte) ;
* chaque `class A` définit implicitement un *type* `A` associé, qui représente l'ensemble de toutes les instances de la classe `A`.

Le type `int | None` représente *l'union* de `int` et `None`.
Ainsi, il est clair que toute valeur de type `int` est aussi de type `int | None`, et il en va de même pour l'unique valeur `None`.
`int` et `None` sont donc des *sous-types* de `int | None`.

Lorsque $S$ est un sous-type de $T$, on peut utiliser une expression de type $S$ là où un $T$ est attendu.
Voici quelques exemples :

```python
def foo(x: int | None) -> str:
    return str(x)

a = 5                   # x inféré de type int
b: int | None = a       # ok, int est un sous-type de int | None
print(foo(a))           # ok, pour la même raison
c: int | None = None    # ok, la valeur None est de type None, qui est sous-type de int | None
d: int | None = "bar"   # erreur : str n'est pas un sous-type de int | None
print(foo("bar"))       # erreur, pour la même raison
e: int | str | None = b # ok, int | None est un sous-type de int | str | None
```

Que peut-on *faire* avec une variable de type `A | B` ?
En général, pas grand chose.
Souvent, cela se limite à trois possibilités :

* les opérations universelles qui s'appliquent à tous les types, telles que `str(x)`, `x is y` ou `x == y` ;
* *tester* le type avec un `match` pour traiter séparément les différents cas :

  ```python
  match x:
      case A():
          # on sait que `x` est un `A` ici
          x.methode_de_A()
      case B():
          x.methode_de_B()
  ```

* dans le cas particulier de `A | None`, le test `x is None`/`x is not None` est un raccourci pour un `match`:

  ```python
  if x is None:
      ... # le cas None, où on n'utilise plus `x`
  else:
      # on sait que `x` est un `A` ici
      x.method_de_A()
  ```

Un cas très concret que nous avons déjà rencontré la semaine passée est celui de la méthode `__eq__`.
Elle doit prendre en paramètre une valeur de type `object`.
Ce type est un peu particulier : c'est l'ensemble de *toutes* les valeurs possibles en Python.
Autrement dit, c'est l'union de tous les types possibles !
L'usage du `match` dans `__eq__` permet d'isoler parmi toutes ces valeurs possibles celles qui sont de la même classe.

```python
class Vec2:
    ...

    def __eq__(self, other: object) -> bool:
        match other:
            case Vec2():
                # On sait que `other` est de type `Vec2` ici.
                return self.x == other.x and self.y == other.y
            case _:
                # Tous les autres cas.
                # La valeur de retour `NotImplemented` est spéciale.
                # Mypy l'accepte malgré que le résultat soit `bool`.
                return NotImplemented
```

### Définitions

**Définition :** Un type $T$ est un membre de l'ensemble de tous les types.

Bon, d'accord, c'est une définition quelque peu auto-réalisatrice.
Ce qu'on veut *vraiment* dire avec cette définition, c'est surtout que, formellement, un type n'est *pas* un ensemble de valeurs.
Il *représente* un ensemble de valeur, dans son intention, mais ce n'est pas la même sorte d'objet mathématique.
Par exemple, `int` *représente* l'ensemble $\mathbb{Z}$, mais il *n'est pas* cet ensemble (`int` $\neq \mathbb{Z}$).

**Définition :** La relation de sous-typage $<:$ est une relation binaire sur les *types*.
On note $A <: B$ et on lit "$A$ [est un] sous-type de $B$".
Pour avoir de bonnes propriétés, chaque relation de sous-typage devrait être *réflexive* et *transitive*, ce qui en fait un *préordre*.

Comme tout préordre, la relation peut s'inverser : on note $B >: A$ et on lit "$B$ [est un] super-type de $A$" si et seulement si $A <: B$.

Les relations de sous-typage sont complexes.
Elles sont définies par un ensemble de *règles* qui forment un *système de types*.

Certains langages (dont TypeScript et Scala) ont des systèmes de types tellement complexes que la transitivité est parfois cassée, ce qui leur pose de réels problèmes.
Dans le cadre de notre étude du sous-typage, on supposera que $<:$ est transitive, et est un préordre.

Voici *quelques unes* des règles de sous-typage de Python :

* `T <: T` pour tout `T` (réflexivité) ;
* `S <: object` pour tout `S` (tous les types sont sous-types de `object`) ;
* `A <: A | B` et `B <: A | B` pour tous `A` et `B` ;
* si `A <: C` et `B <: C`, alors `A | B <: C`.

Remarquez que si l'on combine les deux dernières règles, on obtient que `int | str <: str | int`, avec `A = int`, `B = str` et `C = str | int`.
En effet, `int <: str | int` et `str <: str | int`.

**Définition :** On note $=:=$ la relation d'équivalence dérivée de $<:$, c'est-à-dire : $A =:= B$ si et seulement si $A <: B \land B <: A$.

Notez qu'il est possible pour deux types $X$ et $Y$ d'être *équivalents* ($X =:= Y$) bien que *différents* ($X \neq Y$).
Par exemple, `int | str` et `str | int` sont équivalents mais différents.
Syntaxiquement, ils sont clairement différents (le "texte" `int | str` n'est pas le même que `str | int`).
Pourtant, `int | str <: str | int` et `str | int <: int | str`.
C'est à cause de ça que $<:$ n'est qu'un *préordre*, et non un *ordre partiel* ; $<:$ n'est pas anti-symétrique.

**Définition :** La relation $x: T$ met en relation une *expression* $x$ et un type $T$.
Elle se lit "$x$ a le type $T$" ou "$x$ est de type $T$".
Cette relation doit satisfaire la propriété de *subsomption* : si $x: S$ et $S <: T$, alors $x: T$ pour tous $x, S, T$.

Dans son *intention*, on veut que $x: T$ lorsque la valeur de $x$ est un élément de l'ensemble de valeurs représenté par $T$.
Par exemple, on a $5: \texttt{int}$, car $5 \in \mathbb{Z}$.

Par la propriété de subsomption, on a donc aussi $5: \texttt{int \| str}$, puisque `int <: int | str`.

## Héritage

L'héritage est une relation entre *classes*.
Lorsque l'on définit une classe `A`, on peut spécifier une (ou plusieurs, comme nous le verrons plus tard dans le semestre) classe de base `B`.
On dit que `A` *étend* `B` ou a `A` *hérite* de `B`.
On dit aussi que `A` est une classe *enfant* de `B` ou une *sous-classe* de `B`.
`B` est une classe *parent* de `A` ou une *super-classe* de `A`.

Une classe `A` définie de la sorte reçoit automatiquement tous les attributs et les méthodes définies dans `B`, d'où l'appellation "héritage".

### Premier exemple

Voici un exemple, qui n'a aucun sens, mais qui illustre la mécanique :

```python
class Parent:
    a: Final[int]

    def __init__(self, a: int) -> None:
        self.a = a

    def foo(self, s: str) -> str:
        return f"{s} {self.a}"

class Child(Parent):
    b: Final[str]

    def __init__(self, a: int, b: str) -> None:
        super().__init__(a)
        self.b = b

    def bar(self, s: str) -> str:
        t = self.foo(s) # on peut utiliser self.foo
        return f"{t} {self.b} {self.a}" # et self.a

def main() -> None:
    child = Child(5, "hello")
    print(child.b) # hello
    print(child.a) # 5 -- a est hérité de Parent
    print(child.foo("world")) # world 5 -- foo est héritée de Parent
    print(child.bar("world")) # world 5 hello 5
```

La syntaxe

```python
class Child(Parent):
```

définit `Child` comme une classe qui *étend* `Parent`.
Elle reçoit donc tous les membres définis dans `Parent` : l'attribut `a` et la méthode `foo`.
D'une certaine manière, on peut considérer que le contenu de `Parent` est "copié" automatiquement dans `Child` (ce n'est pas la vraie mécanique, mais ça peut aider pour l'instant).

L'initialisation *via* la méthode spéciale `__init__` devient compliquée.
Comment s'assurer que les instances de `Child` sont bien *entièrement* initialisées, y compris pour les attributs définis dans `Parent` ?
La seule manière est de faire en sorte que `Parent.__init__` soit bien appelée sur l'instance `c` de `Child`.
C'est ce que fait l'instruction

```python
super().__init__(a)
```

La mécanique complète des "appels à `super`" est complexe.
Pour l'instant, nous utilisons une explication simplifiée.
Lors de cet appel, les étapes suivantes ont lieu :

1. Observer qu'on se trouve lexicalement dans la classe `Child`, qui étend `Parent`.
2. Trouver la méthode `__init__` déclarée dans `Parent`.
3. Appeler cette méthode avec `self` (le premier argument de la méthode `__init__` en cours) et `a` en arguments.

Concrètement, on appelle donc bien `Parent.__init__` avec `self` et `a` en arguments.
Cela permet d'effectuer l'initialisation de `Parent` avant de celle de `Child`.

*Toutes* les classes qui en étendent une autre doivent appeler `super().__init__`.

### Héritage et sous-typage

Rappelez-vous que chaque définition de classe `C` définit un *type* `C` associé.
La relation d'héritage établit également une relation de *sous-typage* correspondante.
Dans l'exemple ci-dessus, nous avons donc la relation `Child <: Parent`.

Cela veut dire que le code suivant est correct :

```python
child = Child(5, "hello")
parent: Parent = child # ok, Child <: Parent
print(parent.a) # 5
print(parent.foo("world")) # world 5
```

Par contre, les instructions suivantes sont rejetées par mypy :

```python
print(parent.b)
print(parent.bar("world"))
```

En effet, `b` et `bar` ne sont pas définies dans `Parent`.
Il existe donc des opérations qu'on peut faire avec `Child`, mais pas avec `Parent`.
En tant que sous-type, `Child` donne plus de garanties que `Parent`.

Il se passe la même chose avec `int` et `int | str`.
Si `x: int`, vous pouvez faire `x + 1`, alors que vous ne pouvez pas le faire avec `x: int | str`.

En revanche, toute opération que l'on peut faire sur `Parent` peut aussi être faite sur `Child`.

### Surcharge et liaison tardive

Il est possible pour une classe enfant de *surcharger* (redéfinir, ou *override* en anglais) des méthodes héritées de sa classe parent.
Prenons un exemple un peu moins artificiel : `Rectangle` et `Square`.

```python
class Rectangle:
    """A rectangle."""

    width: Final[int]
    height: Final[int]

    def __init__(self, width: int, height: int) -> None:
        self.width = width
        self.height = height

    def area(self) -> int:
        """Area of the rectangle, which is `width * height`."""
        return self.width * self.height

class Square(Rectangle):
    """A square is a rectangle with equal width and height."""

    def __init__(self, side: int) -> None:
        super().__init__(side, side)

    @property
    def side(self) -> int:
        """Returns the (unique) side length."""
        return self.width

    # Let's pretend that computing `side ** 2` is faster than
    # `width * height`. We might then want to compute the area faster for
    # squares with an override.
    def area(self) -> int:
        return self.side ** 2
```

Remarquez que `Square` redéfinit `area`, qui existait déjà dans `Rectangle`.
Dans ce cas, la méthode `area` de `Square` a priorité pour les instances de `Square`.

```python
square = Square(5)
a = square.area() # appelle Square.area, pas Rectangle.area
```

La méthode choisie dépend de la *classe* à laquelle l'instance appartient à l'exécution.
Elle est indépendante du *type* perçu de la variable.
Le code suivant appelle *aussi* `Square.area` :

```python
square = Square(5)
rectangle: Rectangle = square # ok, Square <: Rectangle
a = rectangle.area() # appelle Square.area
```

C'est ce qu'on appelle le principe de *liaison tardive*.
Ce n'est pas le type de la variable (connu "tôt", lors de la compilation) qui détermine la méthode appelée, mais bien la classe de l'instance (connue "tard", lors de l'exécution du programme).

Utilisez le débogueur pour vous convaincre que c'est bien ce comportement qui est appliqué.
Vous pouvez aussi ajouter des `print()` dans les deux méthodes.

## Polymorphisme

La combinaison du sous-typage et de la liaison tardive apporte une technique de développement extrêmement puissante : le polymorphisme par sous-typage.
C'est cette technique qui a rendu la programmation orientée objet importante.
Sans cette combinaison, on aurait pu continuer à développer avec des `struct`s et des fonctions globales.

Le mot polymorphe vient des mots grecs πολλοί (plusieurs) et μορφή (la forme).
Une variable polymorphe est donc une variable qui peut prendre plusieurs formes.
Par forme, on entend ici la *classe* de l'instance.

Imaginons une fonction qui compare deux `Rectangle` par leur aire :

```python
def biggest_rectangle(x: Rectangle, y: Rectangle) -> Rectangle:
    if x.area() > y.area():
        return x
    else:
        return y
```

Grâce à la relation de sous-typage entre `Square` et `Rectangle`, il est possible d'appeler cette fonction avec des instances des deux classes.
Les quatre combinaisons sont possibles:

```python
biggest_rectangle(Rectangle(5, 7), Rectangle(4, 8))
biggest_rectangle(Rectangle(5, 7), Square(6))
biggest_rectangle(Square(6), Rectangle(4, 8))
biggest_rectangle(Square(5), Square(6))
```

À l'exécution, les paramètres `x` et `y` peuvent prendre plusieurs formes : soit des instances de `Square`, soit des instances de `Rectangle`.
À cause de la liaison tardive, on ne peut pas savoir, *a priori*, quelle version de `area()` sera appelée avec `x.area()` et `y.area()`.

C'est une grande force !
Cela veut dire qu'avec un code *unique*, `biggest_rectangle` est capable de traiter 4 combinaisons de formes différentes.
Le polymorphisme permet de traiter de manière *uniforme* des instances de classes différentes.

Avec des `struct`s et des fonctions globales, il aurait été *impossible* d'exprimer ce degré de polymorphisme de manière directe.
Nous aurions eu besoin de 4 fonctions différentes.

### Les dangers du polymorphisme

Le grand pouvoir que nous donne le polymorphisme s'accompagne, comme le dit le proverbe, de grandes responsabilités.
Il est important de ne pas définir des pseudo-relations de sous-typage, au travers de l'héritage, qui casseraient les propriétés importantes des classes.

Une propriété fondamentale de notre `Rectangle`, qui est même documentée dans sa méthode `area()`, est que pour tout `r: Rectangle`, on a `r.area() == r.width * r.height`.
Ceci est vrai également si `r` est en fait un `Square`.
En effet, la méthode `Square.area()` respecte bien ce contrat, car la classe `Square` garantit en interne que `s.side == s.width == s.height` pour tout `s: Square`.
Jusque là, tout va bien.
On peut écrire du code qui manipule des `Rectangle` et qui se repose sur cette garantie.
On peut réifier cette garantie avec un `assert`, par exemple :

```python
r: Rectangle = ...
assert r.area() == r.width * r.height
```

Modifions maintenant un tout petit peu notre classe `Rectangle` :

```diff
 class Rectangle:
     """A rectangle."""

-    width: Final[int]
-    height: Final[int]
+    width: int
+    height: int

     ...
```

Prenez quelques dizaines de secondes et essayez de trouver le problème que cela pose.

⋮

⋮

⋮

⋮

⋮

⋮

On peut casser les garanties de `Rectangle.area` avec, par exemple, le code suivant :

```python
def test_rectangle() -> None:
    r: Rectangle = Square(5)
    r.height = 7
    assert r.area() == r.width * r.height
```

```
    def test_rectangle() -> None:
        r: Rectangle = Square(5)
        r.height = 7
>       assert r.area() == r.width * r.height
E       assert 25 == (5 * 7)
E        +  where 25 = area()
E        +    where area = <main.Square object at 0x7efc5cb296a0>.area
E        +  and   5 = <main.Square object at 0x7efc5cb296a0>.width
E        +  and   7 = <main.Square object at 0x7efc5cb296a0>.height
```

Cette problématique nous amène au principe de substitution de Liskov.

### Principe de substitution de Liskov

En anglais : the [*Liskov Substitution Principle*](https://en.wikipedia.org/wiki/Liskov_substitution_principle), ou LSP.

Le principe de substitution de Liskov donne une règle générale pour décider s'il est admissible que $A <: B$.
À l'origine, il a été énoncé par Barbara Liskov comme une *définition* du sous-typage :

> If for each object $o_1$ of type $S$ there is an object $o_2$ of type $T$ such that for all programs $P$ defined in terms of $T$, the behavior of $P$ is unchanged when $o_1$ is substituted for $o_2$, then $S$ is a subtype of $T$.

En formule et dans sa version moderne, elle donne ceci.
Pour tout prédicat $\phi$,

$$ S <: T \rightarrow (\forall o_2: T. \phi(o_2) \rightarrow \forall o_1: S. \phi(o_1)) $$

Dans cette formulation originale, $o_2: T$ n'inclut *pas* les instances d'une "sous-classe" potentielle de $T$.
Les sous-classes n'induisent pas une relation de sous-typage *a priori*.

Dans la vie de tous les jours, on pense au LSP de la façon suivante : si $S <: T$, alors tout ce qu'on peut faire avec une valeur de type $T$, on doit aussi pouvoir le faire avec une valeur de type $S$.
Autrement dit, on peut *substituer* un $T$ par un $S$ dans tout programme sans casser ce programme.

Instancions ce principe à notre problème des `Rectangle` et `Square` :

* $S$ est `Square` ;
* $T$ est `Rectangle` ;
* $\phi(o)$ est `o.area() == o.width * o.height`.

On peut voir que nous pouvons effectivement prouver que $\forall o_2: T. \phi(o_2)$.
En effet, pour toute instance $o_2$ de $T =$ `Rectangle` (directe, donc créée avec `Rectangle(...)`), on peut prouver $\phi(o_2)$.
Par contre, nous avons trouvé une instance $o_1$ de $S =$ `Square` pour laquelle $\lnot \phi(o_1)$.
Il n'est donc pas vrai que $\forall o_1: S. \phi(o_1)$.

Pour ces choix de $S$, $T$ et $\phi(o)$, la partie conséquente de l'implication du LSP est *fausse* :

$$ \forall o_2: T. \phi(o_2) \rightarrow \forall o_1: S. \phi(o_1) $$

Par contraposée, il faut que $S <: T$ soit *faux* également.
Or, notre définition de `Square` en Python, avec de l'héritage, a induit par construction que $S <: T$.
Nous avons donc violé le principe de subsitution de Liskov !

Apparemment, Barbara Liskov nous dit **qu'un carré ne peut pas être un rectangle** !
Je ne sais pas vous, mais ce n'est pas ce que j'ai appris à l'école primaire.
Qui a raison ?
Liskov, ou la géométrie élémentaire ?

Sans surprise, la réponse est : *les deux*.
Car nous avons détourné la définition même de "rectangle" en rendant ses attributs muables.
Ce que nous dit vraiment Liskov, c'est qu'un carré ne peut pas être un rectangle *muable*.
La géométrie, elle, nous dit qu'un carré *immuable* est un rectangle *immuable*.

Effectivement, si on retourne à notre définition de `Rectangle` dont les attributs étaient `Final`, on ne peut pas trouver de contradiction au LSP.
Vous n'avez pas les outils mathématiques pour le *prouver* (et ça ne fait généralement partie que du cursus en IC), mais vous devriez pouvoir vous en convaincre.

### Réparer LSP pour nos classes muables

Vous avez peut-être déjà compris qu'il est beaucoup plus facile de respecter le LSP avec des classes *immuables*.
C'est une des raisons pour lesquelles je vous encourage depuis le départ à rendre vos classes immuables dès que l'occasion se présente.
Plus généralement, c'est la discipline recommandée dans le monde du développement logiciel.
Toute classe qui *peut* être immuable devrait l'être.

Pour les situations où nous avons besoin de classes muables, il nous faut d'autres stratégies pour respecter le LSP.

La stratégie la plus commune consiste à **éviter purement et simplement l'héritage**.
À la place, on privilégie la *composition* : plutôt que définir `Square` comme *sous-classe* de `Rectangle` (ce qui implique que tout `Square` *est* un `Rectangle`), on va *utiliser* un `Rectangle` dans *l'implémentation privée* de `Square`.
C'est de l'encapsulation !

```python
class Square:
    """A square."""

    __rectangle: Final[Rectangle]
    """The internal rectangle. Always has `width == height`."""

    def __init__(self, side: int) -> None:
        self.__rectangle = Rectangle(side, side)

    @property
    def side(self) -> int:
        """Returns the (unique) side length."""
        return self.__rectangle.width

    @side.setter
    def side(self, new_side: int) -> None:
        # Set both width and height to preserve the property
        self.__rectangle.width = new_side
        self.__rectangle.height = new_side

    def area(self) -> int:
        """Area of the rectangle, which is `size ** 2`."""
        return self.side ** 2
```

En cachant le rectangle comme détail d'implémentation, la classe `Square` empêche ses utilisateurs de redéfinir `width` séparément de `height`.
Elle peut donc s'assurer de protéger ses garanties internes, et donc son interface.

> Évidemment, dans cet exemple précis, il vaudrait encore mieux n'avoir qu'un `__side: int`, et pas de rectangle du tout.
> On illustre la méthodologie ici, qui sera plus utile dans des cas plus complexes.

Le problème, en se débarrassant de l'héritage, est qu'on a aussi perdu le polymorphisme.
On ne peut plus utiliser notre fonction `biggest_rectangle` avec des `Square`.

Peut-on avoir le beurre et l'argent du beurre ?
Oui !
Grâce aux classes et méthodes abstraites.

### Classes et méthodes abstraites

⚠️ On ne parle pas ici du concept de "abstract base class" de Python, bien qu'il soit lié.
Si vous cherchez sur le Web (ou avec votre IA favorite) des informations sur les classes et méthodes abstraites en Python, vous risquez beaucoup de tomber sur les "abstract base classes" (ABC).
Nous reparlerons des ABC plus tard dans le semestre.
Cette semaine, nous voyons un concept *plus simple*, mais qui est très peu documenté sur le Web *avec Python*. ⚠️

Reprenons la méthode `biggest_rectangle` :

```python
def biggest_rectangle(x: Rectangle, y: Rectangle) -> Rectangle:
    if x.area() > y.area():
        return x
    else:
        return y
```

À bien y regarder, cette méthode n'a pas besoin de toutes les fonctionnalités de `Rectangle`.
Tout ce qui l'intéresse, c'est sa méthode `area()`.
On n'utilise pas `width` ni `height`.
La classe `Square` a aussi une méthode `area()`, mais on ne peut plus utiliser `biggest_rectangle` avec un `Square`, car `Square <: Rectangle` n'est plus vrai.

La solution ?
Introduisons un type *plus général*, commun à `Square` et `Rectangle`, qui supporterait la méthode `area`.
On pourrait appeler ce type `Shape`.
On aurait donc

```python
def biggest_shape(x: Shape, y: Shape) -> Shape:
    if x.area() > y.area():
        return x
    else:
        return y
```

Et une tentative de la définition de `Shape` :

```python
class Shape:
    """A shape with an area."""

    def area(self) -> int:
        """Area of the shape."""
        ... # problème : que mettre ici ?

class Rectangle(Shape):
    ... # comme avant

class Square(Shape):
    ... # comme avant
```

On a un problème sur la défintion de `Shape.area` cependant.
Si on ne la définit pas du tout, on ne peut toujours pas faire `x.area()` dans `biggest_shape`.
Mais si on la définit, on ne sait pas quoi mettre dans son implémentation !

La solution est d'en faire **une méthode abstraite**, en l'annotant avec `@abstractmethod` :

```python
from abc import abstractmethod

class Shape:
    """A shape with an area.

    Shape is an abstract class. It cannot be directly instantiated.
    """

    @abstractmethod
    def area(self) -> int:
        """Area of the shape."""
        ... # This is actually valid Python code!
```

L'implémentation d'une méthode abstraite pour être, littéralement, `...`.
Mypy considère alors que cette méthode est exclusivement une *interface*, sans implémentation associée.

Notre exemple complet fonctionne désormais.
Il est *statiquement* possible d'appeler `x.area()`, puisque l'interface de `Shape` définit bien une cette méthode.
Et lors de *l'exécution*, la liaison tardive fera en sorte que ce soit bien les réelles méthodes `Rectangle.area` ou `Square.area` qui seront appelées.

*Quid* si l'on essaye d'instancier *directement* `Shape` ?

```python
s = Shape()
print(s.area()) # oops ?
```

Dans ce cas, mypy nous sauve, en nous disant qu'il n'est pas possible d'instancier `Shape` car c'est une classe abstraite :

> Cannot instantiate abstract class `Shape` with abstract attribute `area`.

Toute classe qui contient au moins une méthode abstraite (ou hérite d'une méthode abstraite sans l'implémenter) est considérée comme **classe abstraite**.
Il n'est pas possible d'instancier une classe abstraite directement.
On ne peut instancier que ses sous-classes qui auront donné une implémentation à toutes ses méthodes abstraites.

Avec tout ça, on a récupéré le polymorphisme.
Mais est-ce qu'on a respecté le LSP ?
Oui, car au niveau de `Shape`, la seule promesse que nous faisons (et que nous pouvons faire), est que `shape.area()` renvoie l'aire de la `Shape`.
La spécification de `Shape.area` est *plus générale* que celles de `Rectangle.area` et `Square.area`.
Ces dernières *raffinent* la spécification de manière compatible, en établissant un lien avec les autres attributs et/ou méthodes de leurs classes respectivent.

## Conclusion

Le polymorphisme est une méthode de développement très puissante.
Il permet d'appliquer des traitements *uniformes* à des objets de formes *différentes*.

En Python, on met en œuvre le polymorphisme (de sous-typage) grâce aux notions de *sous-typage* et de *liaison tardive*.
On exploite *l'héritage* comme moyen pour définir de nouveaux sous-types.

Afin de profiter pleinement du polymorphisme, il est important de bien définir les *interfaces* des types polymorphes.
Lorsqu'on les implémentes, il est crucial de respecter le principe de substitution de Liskov (Liskov Substitution Principle, LSP).

### Mieux comprendre votre projet

Dans votre projet, vous avez dû définir votre `GameView` avec plusieurs aspects "magiques" :

```python
class GameView(arcade.View):
    """Main in-game view."""

    def __init__(self) -> None:
        # Magical incantion: initialize the Arcade view
        super().__init__()

        # Choose a nice comfy background color - but who defined that attribute?
        self.background_color = arcade.csscolor.CORNFLOWER_BLUE

        # Setup our game
        self.setup()

    # Who calls that method?
    def on_draw(self) -> None:
        """Render the screen."""
        self.clear()
        self.player_sprite_list.draw()

def main() -> None:
    """Main function."""

    # Create the (unique) Window, setup our GameView, and launch
    window = arcade.Window(WINDOW_WIDTH, WINDOW_HEIGHT, WINDOW_TITLE)
    game_view = GameView()
    window.show_view(game_view) # Why can I give `game_view` here?
    arcade.run()
```

Vous avez maintenant les concepts nécessaires pour réellement comprendre tous ces petits détails :

* L'en-tête `class GameView(arcade.View):` définit `GameView` comme héritant de `arcade.View`.
  Cela induit la relation de sous-typage `GameView <: arcade.View`.
* L'appel `super().__init__()` appelle la méthode `__init__` de `arcade.View`, avec `self` comme unique argument.
* La propriété `self.background_color` existe car elle est déclarée dans `arcade.View`, et que vous la recevez par héritage.
* Vous pouvez passer `game_view` en argument de `window.show_view`, qui attend une `arcade.View` en paramètre, grâce à la relation de sous-typage `GameView <: arcade.View`.
* L'implémentation d'Arcade, quelque part, peut appeler votre méthode `GameView.on_draw` par *liaison tardive*.
  Le fonctionnement interne de `arcade.Window` exploite le *polymorphisme* sur `arcade.View`.

A-t-on respecté le LSP dans tout ça ?
Oui, car la spécification de `arcade.View` est suffisamment générique.
Elle ne donne pas trop de garanties sur ce que sa méthode `on_draw` fait *exactement*.
Elle dit seulement qu'elle doit fournir votre "custom drawing code".

Il en va de même pour les autres `on_*` que vous avez implémentées.

La classe `arcade.Sprite` est assez générique également.
Qui sait ?
Peut-être que vous pourriez avoir envie de définir vos propres sous-classes de `arcade.Sprite`.
