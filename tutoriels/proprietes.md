---
title: Propriétés
layout: default
---

# Propriétés

Nous avons déjà vu que les classes Python peuvent contenir des *attributs* et des *méthodes*.
Les attributs sont des champs dont la valeur est stockée dans l'instance, auxquels on accède *via* une syntaxe `x.foo`.
Les méthodes sont des *opérations* appliquées à l'instance, sans stockage associé, que l'on appelle *via* une syntaxe `x.foo(...)`.

Dans ce court tutoriel, nous découvrons la troisième et dernière sorte de membre des classes : les *propriétés*.
Celles-ci sont un hybride entre attributs et méthodes.
De l'extérieur, elles ressemblent à des attributs : on y accède sous la forme `x.foo`.
Mais à l'intérieur de la classe, elles sont implémentées avec des méthodes : elles sont calculées par des opérations plutôt que d'être nécessairement stockées.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Proriétés en lecture seule

Reprenons l'exemple de la class `Person` que nous avions vue [dans notre introduction aux méthodes](./classes.html#méthodes).

```python
class Person:
    first_name: str
    last_name: str

    def __init__(self, first_name: str, last_name: str) -> None:
        self.first_name = first_name
        self.last_name = last_name

    def full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"

p = Person("Arthur", "Weasley")
print(p.full_name()) # affiche "Arthur Weasley"
```

Dans cet exemple, nous avions défini `full_name` comme une *méthode*.
Nous voulions calculer sa valeur d'après les attributs `first_name` et `last_name`, sans stocker une copie de l'information.
Cela nous force cependant à *utiliser* cette méthode sous la forme `p.full_name()`.

Pourtant, intuitivement, du point de vue du code utilisateur de la classe, on ne s'attend pas à ce que `full_name` *fasse* quelque chose, c'est-à-dire à ce qu'elle soit une *opération*.
Il semble qu'on devrait pouvoir lire cette information *comme si* c'était un attribut, sous la forme `p.full_name`.

C'est précisément le rôle d'une *proprité* (*property*).
Une propriété se "fait passer pour" un attribut de l'extérieur, mais est implémentée par une méthode.
La définition d'une propriété se fait en annotant la méthode avec `@property` :

```diff
+    @property
     def full_name(self) -> str:
         return f"{self.first_name} {self.last_name}"

 p = Person("Arthur", "Weasley")
-print(p.full_name()) # affiche "Arthur Weasley"
+print(p.full_name) # affiche "Arthur Weasley"
```

### Principe d'accès uniforme

Les propriétés sont une *mécanique* du langage Python.
Elles répondent à un principe général appliqué dans beaucoup de langages de proprammation : le *principe d'accès uniforme*, ou *uniform access principle*.
Ce principe a été énoncé pour la première fois par Bertrand Meyer :

> All services offered by a module should be available through a uniform notation, which does not betray whether they are implemented through storage or through computation.

Tranduit en français :

> Tous les services offerts par un module devraient être disponibles *via* une notation uniforme, qui ne laisse pas transparaître s'ils sont implémentés par du stockage ou par une opération.

Le concept de "module" de ce principe se traduisant en "classe" dans notre incarnation en Python.

Vous retrouverez ce principe dans de nombreux langages :

* Scala : `val fullName: String` ou `def fullName: String`
* JavaScript : `fullName: "..."` ou `get fullName() { ... }`
* Pascal Objet / Delphi : `FullName: String` ou `property FullName: String`
* etc.

Dans toutes ces incarnations du principe d'accès uniforme, on accéderait aux attributs comme aux propriétés avec la notation `instance.fullName`.

Parmi les langages principaux qui ne supportent *pas* le principe d'accès uniforme, on trouve C++ et Java.

### Exposer un "attribut" en lecture seule

On peut se servir de `@property` pour exposer un attribut en lecture seule au code utilisateur d'une classe.
Par exemple, même si l'on veut exposer `first_name` et `last_name` directement en lecture, on ne veut sans doute pas laisser n'importe qui *modifier* ces champs.
On peut obtenir cet effet en *stockant* ces attributs [de manière *privée*](./classes.html#méthodes-et-encapsulation), et en exposant une propriété publique pour les lire.

```python
class Person:
    __first_name: str
    __last_name: str

    def __init__(self, first_name: str, last_name: str) -> None:
        self.__first_name = first_name
        self.__last_name = last_name

    @property
    def first_name(self) -> str:
        return self.__first_name

    @property
    def last_name(self) -> str:
        return self.__last_name

    @property
    def full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"

p = Person("Arthur", "Weasley")
print(p.first_name) # affiche "Arthur"
p.first_name = "Percy" # AttributeError: property 'first_name' of 'Person' object has no setter
```

La dernière ligne est rapportée par mypy comme contenant une erreur.
L'exception `AttributeError` est levée si on exécute quand même le programme.

Les propriétés permettent donc d'assurer *l'encapsulation* tout en exposant une *interface* avec un *accès uniforme*.

### Attribut immuable avec `Final`

En utilisant une propriété comme ci-dessus, on obtient un attribut en *lecture seule* depuis *l'extérieur* de la classe.
Cependant, on n'a pas de garantie que les méthodes de la classe ne modifient pas l'attribut.
Ce n'est généralement pas un problème, puisque par définition nous faisons *confiance* au reste de la classe pour préserver sa propre encapsulation.

Il existe cependant un moyen moins verbeux qui garantit *l'immuabilité*, même au sein de la classe : le pseudo-type `Final`.
Lorsqu'un attribut est marqué d'un type `Final`, il ne peut être assigné que depuis le constructeur, et il *doit* y être assigné une et une seule fois.
Voici notre exemple précédent avec des attributs finaux.

```python
from typing import Final

class Person:
    first_name: Final[str]
    last_name: Final[str]

    def __init__(self, first_name: str, last_name: str) -> None:
        self.first_name = first_name
        self.last_name = last_name
        self.first_name = "oops" # Cannot assign to final attribute "first_name"

    def try_to_set_first_name(self, first_name: str) -> None:
        self.first_name = first_name # Cannot assign to final attribute "first_name"

p = Person("Arthur", "Weasley")
print(p.first_name)
p.first_name = "Percy" # Cannot assign to final attribute "first_name"
```

Il est recommandé de déclarer `Final` tout attribut qui ne doit jamais être modifié.
Cela représente la grande majorité des attributs dans un programme habituel.

Les propriétés en lecture seule cachant un champ privé restent très utiles pour les cas où les méthodes *de la classe* ont le droit de modifier la propriété, mais pas l'extérieur.

## Propriété en lecture-écriture

On peut définir une propriété en lecture-écriture.
Cela veut dire que l'écriture doit stocker quelque chose, d'une manière ou d'une autre.
Il y a trois raisons communes pour vouloir une propriété en lecture-écriture :

* Avec un stockage privé direct, mais l'écriture vérifie que la valeur stockée est valide.
* Sans stockage direct, comme propriété *dérivée* d'un autre attribut ; il faut alors faire la transformation inverse à l'écriture.
* Avec un stockage privé direct, mais on souhaite que l'écriture mette aussi à jour d'autres états dépendants.

### Vérification de la validité à l'écriture

Supposons qu'on ait une classe `Rectangle` avec une largeur et une hauteur.
À la création, on s'assure que la largeur et la hauteur sont strictement positives.

```python
class Rectangle:
    width: int
    height: int

    def __init__(self, width: int, height: int) -> None:
        if width <= 0 or height <= 0:
            raise ValueError("Not a valid Rectangle")
        self.width = width
        self.height
```

Dans cette exemple, on suppose qu'on veut laisser le code utilisateur de la classe modifier ces propriétés.
C'est donc un rectangle muable.
Cependant, sans restriction, quelqu'un pourrait modifier `rectangle.width = -2` après `__init__`.
Notre garantie, vérifiée dans le constructeur, serait alors contournée.

On peut utiliser des attributs privés et des propriétés en lecture-écriture pour préserver ces garanties et l'encapsulation.
Une propriété en lecture-écriture se définit avec un *setter*.

```python
class Rectangle:
    __width: int
    __height: int

    def __init__(self, width: int, height: int) -> None:
        if width <= 0 or height <= 0:
            raise ValueError("Not a valid Rectangle")
        self.__width = width
        self.__height = height

    @property
    def width(self) -> int:
        return self.__width

    @width.setter
    def width(self, new_width: int) -> None:
        if new_width <= 0:
            raise ValueError("Width must be positive")
        self.__width = new_width

    @property
    def height(self) -> int:
        return self.__height

    @height.setter
    def height(self, new_height: int) -> None:
        if new_height <= 0:
            raise ValueError("Height must be positive")
        self.__height = new_height

  r = Rectangle(4, 6)
  r.width = 10 # ok
  print(r.width) # 10
  r.height = -2 # ValueError
```

Nous avons ainsi tous les points positifs en même temps :

* Une *interface* propre, suivant le principe d'accès uniforme ;
* Une *encapsulation* forte, où la classe protège ses garanties.

Un setter doit avoir le même nom que le "getter" associé.
Il doit prendre un paramètre en dehors de `self`, avec la valeur qui sera à la droite du `=`.
Il doit en principe renvoyer `None`.

### Propriétés dérivées en lecture-écriture

Imaginons que notre `Rectangle` possède aussi un angle de rotation.
Selon les usages, cet angle sera plus pratique à manipuler en radians ou en degrés.
Pour la plupart des calculs, par exemple, il serait plus pratique d'y accéder en radians, puisque les fonctions mathématiques manipulent des radians.
Par contre, pour manipuler le rectangle d'un point de vue géométrique, il peut être plus facile d'utiliser les degrés.

On ne veut évidemment pas *stocker* l'angle de rotation à la fois en degrés et en radians.
On peut dériver les degrés des radians, et inversement.
On peut utiliser une propriété en lecture-écriture pour cela : stocker les radians, mais proposer une propriété pour les degrés.

```python
import math

class Rectangle:
    ... # width, height

    rotation: float
    """Rotation angle in radians."""

    def __init__(self, width: int, height: int) -> None:
        ... # comme avant
        self.rotation = 0.0

    ... # properties for width, height

    @property
    def rotation_degrees(self) -> float:
        """Rotation angle in degrees."""
        return math.degrees(self.rotation)

    @rotation_degrees.setter
    def rotation_degrees(self, degrees: float) -> None:
        self.rotation = math.radians(degrees)

r = Rectangle(4, 6)
r.rotation_degrees = 90
print(r.rotation) # 1.5707963267948966
```

### Propriétés en lecture-écriture avec mises à jour d'état

Nous avons vu qu'on pouvait éviter de stocker `full_name` en la recalculant à chaque fois à partir de `first_name` et `last_name`.
Dans ce cas, le calcul est simple et rapide.
Imaginons que ce ne soit pas le cas, et que calculer `full_name` soit coûteux.
On pourrait alors vouloir stocker `full_name` (bien que sa valeur soit techniquement redondante), et la mettre à jour à chaque fois que `first_name` ou `last_name` est modifiée.

On peut faire cela avec des propriétés également.
Cette variante est moins fréquente, mais peut avoir son utilité.

```python
class Person:
    __first_name: str
    __last_name: str
    __full_name: str # computed from the other two

    def __init__(self, first_name: str, last_name: str) -> None:
        self.__first_name = first_name
        self.__last_name = last_name
        self.__update_full_name()

    def __update_full_name(self) -> None:
        self.__full_name = f"{self.__first_name} {self.__last_name}"

    @property
    def first_name(self) -> str:
        return self.__first_name

    @first_name.setter
    def first_name(self, first_name: str) -> None:
        self.__first_name = first_name
        self.__update_full_name()

    @property
    def last_name(self) -> str:
        return self.__last_name

    @last_name.setter
    def last_name(self, last_name: str) -> None:
        self.__last_name = last_name
        self.__update_full_name()

    @property
    def full_name(self) -> str:
        return self.__full_name

p = Person("Arthur", "Weasley")
print(p.full_name) # Arthur Weasley
p.first_name = "Percy"
print(p.full_name) # Percy Weasley
```
