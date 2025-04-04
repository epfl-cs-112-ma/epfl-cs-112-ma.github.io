---
title: Fonctions variadiques
layout: default
use_mathjax: true
---

# Fonctions variadiques

Ce chapitre introduit les fonctions variadiques, c'est-à-dire les fonctions qui acceptent un nombre variable d'arguments.
Avec elles, nous découvrons aussi quelques spécificités des signatures de fonctions en Python concernant les paramètres *positionnel* et les paramètres par *mots-clef*.

**Ce chapitre ne fait pas partie de la "matière d'examen".**
De la même manière que les chapitres sur git, j'estime qu'il est nécessaire de vous enseigner ce sujet, mais je ne vous évaluerai pas dessus.

De plus, il n'est pas forcément nécessaire d'intégrer ce contenu lors de la semaine 8.
Vous pouvez probablement le remettre à plus tard.
Il apporte cependant un éclairage supplémentaire au chapitre sur le [pattern matching](./pattern-matching.html).

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Rappel : paramètres optionnels

Vous connaissez déjà les paramètres *optionnels*.
Les paramètres qui sont déclarés avec une valeur par défaut peuvent être omis lors de l'appel d'une fonction.
Par exemple, la fonction [`round()`](https://docs.python.org/3/library/functions.html#round) de Python accepte 1 ou 2 arguments :

```python
>>> round(22.675) # arrondit à l'unité près
23
>>> round(22.675, 2) # arrondit au centième près
22.68
>>> round(22.675, -1) # arrondit à la dizaine près
20.0
```

Sa signature est la suivante (en trichant un tout petit peu ; la vraie est plus compliquée que ça) :

```python
def round(number: float, ndigits: int = 0) -> float:
    ...
```

## Paramètres positionnels et par mots-clef

Vous avez déjà remarqué que, en Python, nous appelons parfois des fonctions en donnant le *nom* des paramètres.
Souvent, nous faisons cela pour des paramètres optionnels.
Avec `arcade.Sprite`, par exemple, qui a plusieurs paramètres optionnels, c'est pratique pour spécifier uniquement ceux qui nous intéressent.

```python
sprite = arcade.Sprite(
    texture,
    scale=0.5,
    angle=90,
)
```

Les arguments ne correspondent pas directement à ceux définis dans la méthode `__init__` de `Sprite`.
Celle-ci est définie comme

```python
    def __init__(
        self,
        path_or_texture: PathOrTexture | None = None,
        scale: float | Point2 = 1.0,
        center_x: float = 0.0,
        center_y: float = 0.0,
        angle: float = 0.0,
    ) -> None:
```

Si on appelait `arcade.Sprite(texture, 0.5, 90)`, le 90 serait la valeur de `center_x`, et non de `angle`.
En spécifiant `angle=90` *par mot-clef* (*keyword argument*), on peut utiliser les valeurs par défaut de `center_x` et `center_y`.

> Souvenez-vous qu'en C++, une telle technique n'est pas possible.
> Il faut donner les arguments de gauche en droite, sans sauter de paramètre.

Pour `scale=0.5`, cela ne change rien pour Python.
Mais pour nous humains, il est beaucoup plus clair de savoir que ce 0.5 se rapporte à la `scale`, et non un autre paramètre potentiel de `Sprite`.
On peut donc aussi les utiliser pour améliorer l'intelligibilité du code, malgré que ce ne soit pas strictement nécessaire.

On peut même utiliser des arguments par mots-clef pour des paramètres *non optionnels*.
Par exemple, on peut appeler la fonction `round` mentionnée ci-dessus avec :

```python
>>> round(number=22.675)
23
>>> round(number=22.675, ndigits=2)
22.68
```

C'est peu utile dans cet exemple, mais cela peut aider si l'odre des paramètres à une fonction n'est pas évident.

On peut aussi *changer l'ordre* des arguments quand on les passe par mots-clef.
L'appel suivant est aussi valide :

```python
>>> round(ndigits=2, number=22.675)
22.68
```

Cela n'est évidemment pas possible lorsque les arguments sont passés par position.

On change rarement l'ordre "exprès".
Si on le fait, c'est plutôt que l'ordre ne semble pas avoir beaucoup d'importance.
Par exemple donner `angle=90` avant `scale=0.5` n'est pas moins bien que dans l'ordre original.
L'usage des arguments par mots-clef permet donc aussi de s'affranchir de l'ordre exact dans lequel sont définis les paramètres.

### Forcer les sortes d'appels

Par défaut, tous les paramètres d'une fonction peuvent être donnés soit de manière positionnelle (sans mot-clef) soit par mot-clef, qu'ils soient optionnels ou non.
On l'a vu avec l'exemple de `round`.
La seule règle est qu'on ne peut pas écrire d'argument positionnel *après* un argument par mot-clef.
L'appel `round(number=22.675, 2)` est donc invalide.

Parfois, lorsqu'on *définit* une fonction, on veut *forcer* les appels à utiliser des arguments positionnels, ou par mots-clef, pour certains paramètres.
Pour ce faire, on utilise les symboles `/` et/ou `*` dans la liste des paramètres.

```python
def foo(pos_only: int, /, pos_or_kw: int, another: str, *, kw_only: str) -> None:
    ...
```

Les arguments `pos_or_kw` et `another` peuvent être donnés soit par position, soit par mot-clef.
L'argument `pos_only`, qui est devant le `/` ne peut être donné que par position.
L'argument `kw_only`, qui est derrière le `*`, ne peut être donné que par mot-clef.

Avec cette définition, les appels suivants sont valides :

```python
foo(1, 2, "bar", kw_only="babar")
foo(1, 2, another="bar", kw_only="babar")
foo(1, pos_or_kw=2, another="bar", kw_only="babar")
```

Par contre, les appels suivants ne le sont pas :

```python
# Too many positional arguments for "foo"
foo(1, 2, "bar", "babar")

# Unexpected keyword argument "pos_only" for "foo"
foo(pos_only=1, pos_or_kw=2, another="bar", kw_only="babar")
```

Si vous voyez des `/` et des `*` dans les signatures des fonctions de Python, vous comprendrez maintenant ce qu'ils veulent dire.

Forcer l'usage de l'un ou l'autre style est contraignant pour les appelants, mais donne plus de libertés à la personne qui définit la fonction pour la changer dans le future :

* Forcer des arguments positionnels laisse la liberté de *changer leur nom* plus tard.
  Puisqu'on sait que `pos_only=1` n'est de toutes façons pas valide, on peut changer son nom sans casser aucun appel.
* Forcer des argument par mots-clef laisse la liberté de *changer leur ordre* plus tard.
  Puisqu'ils doivent toujours être donnés par mots-clef, leur ordre à l'appel n'a pas d'importance.

**Self-check** : à votre avis, est-il possible de définir une fonction avec un `*` avant un `/` ?

### Comparaison avec d'autres langages

Peu de langages permettent au code appelant d'utiliser des arguments nommés.
Deux exceptions sont Scala et C#.
Ces deux langages permettent d'utiliser des arguments nommés

* soit après tous les arguments positionnels (comme Python) ;
* soit avant d'autres arguments positionnels, *seulement si* ils sont "à la bonne place" (c'est-à-dire que retirer le nom ne changerait rien).

```csharp
// C#
Foo(1, 2, another: "bar");
Foo(1, someArg: 2, "bar");
```

```scala
// Scala
foo(1, 2, another = "bar")
foo(1, someArg = 2, "bar")
```

La possibilité de *forcer* les appelants à utiliser soit des arguments nommés, soit par mots-clef, est encore plus rare.
À ma connaissance, seul Ruby rentre dans cette catégorie.
Mais en Ruby, c'est *toujours* forcé : il y a des paramètres positionnels, et des paramètres par mots-clef.
Il n'y a pas de paramètres pour lesquels on peut choisir.

## Paramètres variadiques positionnels

Pour certaines fonctions, on voudrait parfois passer autant d'arguments qu'on le souhaite.

Un exemple classique est la fonction mathématique `hypot`, qui calcule la longueur de l'hypothénuse d'un triangle rectangle.
On pourrait l'écrire comme ceci :

```python
def hypot(x: float, y: float) -> float:
    return math.sqrt(x**2 + y**2)
```

On peut aussi voir cette fonction comme calculant la distance à l'origine d'un point dans l'espace 2D.
Dans cette optique, on pourrait aussi vouloir l'utiliser pour un point dans l'espace 3D.
Il faudrait alors écrire :

```python
def hypot(x: float, y: float, z: float = 0.0) -> float:
    return math.sqrt(x**2 + y**2 + z**2)
```

> La valeur par défaut pour `z` nous permet d'appeler cette fonction avec 2 ou 3 arguments.
> Avec 2 arguments, la valeur de `0.0` correspond à la distance dans le plan.

En général, cela fonctionne aussi en $N$ dimensions.
Mais on ne peut ajouter "une infinité" de paramètres avec des valeurs par défaut.
On voudrait donc plutôt prendre une `Sequence[float]` en argument :

```python
def hypot(coords: Sequence[float]) -> float:
    return math.sqrt(sum([x**2 for x in coords]))
```

On peut maintenant appeler `hypot` avec "autant d'arguments" qu'on le souhaite.
Le seul hic, c'est qu'il faut les mettre dans une `Sequence`, comme une `list`.
Pour les cas communs à 2 et 3 dimensions, cela n'est pas idéal : il faut appeler `hypot([x, y])` au lieu de `hypot(x, y)`.

Peut-on garder l'aspect pratique de `hypot(x, y)`, tout en permettant un nombre arbitraire d'arguments ?
C'est ici que les *fonctions variadiques* entrent en jeu.
On peut définir une telle `hypot` avec la syntaxe suivante :

```python
def hypot(*coords: float) -> float:
    return math.sqrt(sum([x**2 for x in coords]))
```

Notez l'étoile `*` devant le nom du paramètre.
Cette syntaxe indique que `hypot` accepte de 0 à plusieurs paramètres, chacun de type `float`.
On dit que `coords` est un *paramètre variadique*, ou *vararg* (*variadic argument*).

On peut maintenant appeler `hypot(1, 2)`, `hypot(3, 4, 5)`, ou même `hypot(9, 8, 7, 6, 5, 4, 3, 2, 1)`.
Techniquement, on peut même l'appeler avec 1 argument (point dans un espace à 1 dimension) ou même 0 (pas beaucoup de sens mathématique, mais la formule calcule `0.0`) :

```python
>>> hypot(1, 2)
2.23606797749979
>>> hypot(3, 4, 5)
7.0710678118654755
>>> hypot(9, 8, 7, 6, 5, 4, 3, 2, 1)
16.881943016134134
>>> hypot(13.5)
13.5
>>> hypot()
0.0
```

À *l'intérieur* de la fonction, quel est le type de `coords` ?
Il ne peut pas être `float`, puisque la variable contient plusieurs `float`s.
En passant notre souris sur son nom, VS Code, *via* mypy, nous apprend qu'il s'agit d'un `tuple[float, ...]`.
Du point de vue de l'intérieur de la fonction, les paramètres variadiques sont donc des tuples.

Il reste une dernière question : si notre code travaille avec des coordonnées dans un espace à $N$ dimensions, où $N$ n'est connu qu'à l'exécution du programme, comment profitons-nous de `hypot` ?
Supposément, nous aurons déjà nos coordonnées sous forme de `list[float]` ou `tuple[float, ...]`.
Mais on ne peut *pas* donner une telle valeur comme (unique) argument de `hypot`, puisque vue de l'extérieur, celle-ci veut des `float`s :

```python
cs = (3, 4, 5)
# Argument 1 to "hypot" has incompatible type "tuple[int, int, int]"; expected "float"
print(hypot(cs))
```

La solution est d'utiliser `*` *à l'appel* également :

```python
print(hypot(*cs)) # 7.0710678118654755
```

Dans un *appel* à une fonction, `*cs` "étale" (*spreads*) les éléments de `cs` comme des arguments séparés.
On peut utiliser pour `cs` n'importe quel sous-type de `Iterable[float]`.
Rappelez-vous que `Iterable[T]` est la classe de base de toutes les collections de `T`.

On peut définir des fonctions avec des paramètres "normaux" et un paramètre variadique :

```python
def bar(x: int, *ys: float, z: str) -> None: ...
```

qui peut être appelée comme

```python
bar(1, 4.5, 5.6, z="foo")
```

Puisque `ys` s'approprie tous les arguments positionnels à partir du deuxième, `z` doit *nécessairement* être passé par mot-clef.
Comparez avec la définition

```python
def bar(x: int, *, z: str) -> None: ...
```

qui force aussi `z` à être passé par mot-clef, mais où il ne peut pas y avoir d'arguments positionnels après `x`.
Dans les deux cas, le `*` indique que tous les paramètres suivants doivent être passés par mots-clef.

### Comparison avec d'autres langages

La plupart des langages offrent la possibilité de définir des paramètres variadiques.
Leur syntaxe diffère légèrement, mais le sens est toujours le même.
Voici quelques exemples.

```javascript
// JavaScript
function foo(x, y, ...zs) {
  // zs est un Array (l'équivalent JS de list)
}
foo(1, "foo", 5, 6, 7);
args = [5, 6, 7];
foo(1, "foo", ...args); // *args en Python
```

```scala
// Scala
def foo(x: Int, y: String, zs: Int*): Int =
  ... // zs est une Seq[Int] (l'équivalent de Sequence[int])
foo(1, "foo", 5, 6, 7)
val args = List(5, 6, 7)
foo(1, "foo", args*) // *args en Python
```

```java
int foo(int x, String y, int zs...) {
    // zs est un int[] (un array)
}
foo(1, "foo", 5, 6, 7);
int[] args = new int[] { 5, 6, 7 };
foo(1, "foo", args); // pas de symbole pour "spread" en Java !
```

## Paramètres variadiques par mots-clef

Les paramètres variadiques positionnels `*args` capturent tous les arguments *positionnels* "en trop".
De la même façon, les paramètres variadiques *par mots-clef* `**kwargs` capturent tous les arguments *par mots-clef" en trop.

```python
def foo(x: int, y: int, **kwargs: int) -> None:
    # kwargs: dict[str, int] à l'intérieur de la fonction
    print(kwargs)

foo(1, y=2, a=3, b=4) # {'a': 3, 'b': 4}
foo(1, a=3, y=2, b=4) # {'a': 3, 'b': 4}
kwargs = {'c': 5, 'd': 6}
foo(1, y=2, **kwargs) # {'c': 5, 'd': 6}
kwargs2 = {'c': 5, 'y': 2, 'd': 6}
foo(1, **kwargs2) # {'c': 5, 'd': 6}
```

À l'appel, la notation `**kwargs2` *étale* les paires clef-valeur dans le dictionnaire `kwargs2`, comme si on les avait donnés à la main.
`kwargs2` doit être un `Mapping[str, T]` pour correspondre à un *paramètre* `**kwargs: T`.

Notez que la présence du paramètre `y` *s'accapare* l'argument par mot-clef `y`.
Ceci est vrai qu'il soit donné expressément (`y=2`) ou qu'il fasse partie d'un argument étalé `**kwargs2`.

Une fonction peut avoir des paramètres variadiques par position *et* par mots-clef.
Dans ce cas, les `*args` doivent être définis avant les `**kwargs`.

Au sein des langages majeurs, la gestion variadique des arguments nommés est une caractéristique unique de Python.
À ma connaissance, il n'existe pas d'autre langage majeur qui permettent ceci.

## Conclusion

En conclusion, nous donnons finalement la syntaxe la plus générale de déclaration d'une fonction en Python.
Ici nous utilisons

* `[<s>]` pour indiquer que `<s>` est *optionnel* ;
* `{<s>}` pour indiquer que `<s>` peut apparaître de 0 à plusieurs fois ;
* `<s1> | <s2>` pour indiquer que soit `<s1>` peut appraître, soit `<s2>` (mais pas les deux).

```python
def foo(
    [self,] # pour les méthodes
    [{pos_only: T,} /,] # paramètres par position uniquement
    {pos_or_kw: T,} # paramètres par position ou par mots-clef
    [
      *args: T | *, # paramètre variadique par position,
                    # ou marqueur de début d'arguments par mots-clef uniquement
      {kw_only: T,} # paramètres par mots-clef uniquement
    ]
    [**kwargs: T,] # paramètre variadique par mots-clef
) -> TR:
    ...
```

Si un `/` est présent, il doit y avoir au moins un paramètre devant lui.
Si un `*` est présent, il doit y avoir au moins un paramètre (normal) après lui.
