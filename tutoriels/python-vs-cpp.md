---
title: Python vs C++
layout: default
---

# Python vs C++

Cette page contient un résumé des différences principales entre Python et C++.
Avec le premier cours *ex cathedra*, elle constitue votre principale ressource pour transférer vers Python vos connaissances du premier semestre.

## Types

| Type C++                                       | Type Python                      | Valeur C++             | Valeur Python        |
|------------------------------------------------|----------------------------------|------------------------|----------------------|
| `int`, `long`, `size_t`                        | `int`<br>(taille arbitraire)     | `5`                    | `5`                  |
| `unsigned int`                                 | N/A                              |                        |                      |
| `double` (`float`)                             | `float`                          | `5.5`                  | `5.5`                |
| `bool`                                         | `bool`                           | `true`                 | `True`               |
| `std::string`                                  | `str`                            | `"hello"`              | `"hello"`, `'hello'` |
| `char`                                         | N/A (`str` de longueur 1)        | `'c'`                  | `"c"`, `'c'`         |
| `std::vector<T>`<br>(ex. : `std::vector<int>`) | `list[T]`<br>(ex. : `list[int]`) | `vector<int> { 2, 4 }` | `[2, 4]`             |
| `void`                                         | `None`                           | N/A                    | `None`               |

Tous ces types sont prédéfinis en Python.
Il n'y a besoin d'aucun `import`, même pour utiliser `str` et `list`.

En Python, les `int` sont toujours signés, et ont une taille arbitrairement grande.
Vous pouvez parfaitement calculer `50!` avec des `int`, par exemple (essayez !).
Évidemment, plus ils deviennent grands, plus ils consomment de mémoire et prennent du temps de calcul, même pour des opérations "élémentaires".
La plupart des programmes n'utiliseront que des "petits" entiers (par exemple tenant sur 64 bits).
Souvent, cette distinction n'est donc pas très importante.

Notez l'usage des crochets `[]`, au lieu des chevrons `<>`, pour spécifier le type d'éléments d'une `list`.

En C++, `void` n'a aucune valeur.
En Python, il existe une vraie valeur de type `None`, qui s'écrit aussi `None`.

## Commentaires

En C++, il y a deux types de commentaires :

```cpp
// commentaire de ligne

/* commentaire
 * sur plusieurs lignes
 */
```

En Python, il n'existe que des commentaires de ligne :

```python
# commentaire de ligne
```

## Définition de fonction

C++ utilise la syntaxe

```cpp
type_de_retour nom_fonction(type_param1 param1, ...) {
  // corps
}
```

Par exemple :

```cpp
void printInt(int x) {
  std::cout << x << std:endl;
}

bool negative(int x) {
  return x < 0;
}

/** Retourne `x + y`.
 *  `y` vaut 1 par défaut.
 */
int inc(int x, int y = 1) {
  return x + y;
}
```

En Python, la syntaxe est

```python
def nom_fonction(param1: type_param1, ...) -> type_de_retour:
    # corps
```

Le corps d'une fonction est indenté de 4 espaces.
C'est l'indentation qui détermine où s'arrête le corps, comme pour tous les autres blocs.

Par exemple:

```python
def print_int(x: int) -> None:
    print(x)

def negative(x: int) -> bool:
    return x < 0

def inc(x: int, y: int = 1) -> int:
    """Retourne `x + y`.

    `y` vaut 1 par défaut.
    """
    return x + y
```

La chaîne entre `"""` s'appelle la *docstring* d'une fonction.
C'est dans cette chaîne que l'on place la documentation/spécification.
Elle s'affiche dans VS Code lorsqu'on passe le curseur sur le nom d'une fonction.

Il n'est pas possible de déclarer plusieurs fonctions avec le même nom en Python.
Il n'y a pas d'*overloading*, comme en C++.

De plus, il n'y a pas de notion de *prototype* de fonction en Python.
On peut utiliser des fonctions définies plus loin dans le fichier sans déclaration explicite.

On peut se demander comment écrire un bloc *vide*, si les blocs sont régis entièrement par l'indentation.
La réponse est : on ne peut pas !
C'est pourquoi Python possède une instruction `pass`, qui ne fait rien :

```python
def do_nothing() -> None:
    pass
```

## Déclaration de variables locales

<table>
<thead><tr><th>C++</th><th>Python</th></tr></thead>
<tbody>
<tr>
<td>
{% highlight cpp %}
int x = 5;
int x(5);
int x { 5 };

auto x = 5; // type inféré
{% endhighlight %}
</td>
<td>
{% highlight python %}
x = 5      # type inféré
x: int = 5 # type explicite
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

Python n'utilise pas de `;` à la fin des instructions.
Chaque nouvelle ligne débute une nouvelle instruction.

En C++, il est habituel de spécifier explicitement le type des variables, surtout pour des types simples.
On n'utilise `auto` que dans relativement peu de situations.
En Python, c'est l'inverse.
Pour les variables locales, on n'utilisera généralement pas de type explicite.
On laisse leurs types être inférés.
On n'utilise un type explicite que pour attirer l'attention sur celui-ci.

Notez que dans VS Code, si vous passez votre souris sur une variable, une bulle d'aide vous montrera son type (inféré ou non).

![Type inféré d'une variable dans VS Code](/assets/img/vs-code-inferred-var-type.png)

## Instructions et expressions élémentaires

<table>
<thead><tr><th>C++</th><th>Python</th></tr></thead>
<tbody>
<tr><th colspan="2">Affichage à l'écran</th></tr>
<tr>
<td>
{% highlight cpp %}
cout << x << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(x)
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
cout << "x vaut " << x << "." << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(f"x vaut {x}.")
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
// sans endl
cout << x;
{% endhighlight %}
</td>
<td>
{% highlight python %}
# explicitement end=""
print(x, end="")
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Arithmétique</th></tr>
<tr>
<td>
{% highlight cpp %}
-a
a + b
a - b
a * b
a % b
{% endhighlight %}
</td>
<td>
{% highlight python %}
-a
a + b
a - b
a * b
a % b
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
// division entière
// implicite par le type int
a / b
{% endhighlight %}
</td>
<td>
{% highlight python %}
# division entière explicite avec //
a // b
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
// division flottante
// implicite par le type double
a / b
{% endhighlight %}
</td>
<td>
{% highlight python %}
# division flottante explicite avec /
a / b
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Opérateurs booléens</th></tr>
<tr>
<td>
{% highlight cpp %}
!a
not a
{% endhighlight %}
</td>
<td>
{% highlight python %}
not a
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
a && b
a and b
{% endhighlight %}
</td>
<td>
{% highlight python %}
a and b
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
a || b
a or b
{% endhighlight %}
</td>
<td>
{% highlight python %}
a or b
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Appels de fonctions</th></tr>
<tr>
<td>
{% highlight cpp %}
// appel de fonction
foo(a, b)
{% endhighlight %}
</td>
<td>
{% highlight python %}
# appel de fonction
foo(a, b)
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">
En Python, on peut explicitement écrire le nom d'un paramètre lors de l'appel.
C'est le plus souvent utilisé pour des paramètres qui ont une valeur par défaut.
</td></tr>
<tr>
<td>
N/A
</td>
<td>
{% highlight python %}
inc(5, y=7)
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Opérations sur les vecteurs/listes et chaînes de caractères ("collections" en général)</th></tr>
<tr>
<td>
{% highlight cpp %}
collection.size()
{% endhighlight %}
</td>
<td>
{% highlight python %}
len(collection)
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
collection[i]
collection.at(i)
{% endhighlight %}
</td>
<td>
{% highlight python %}
collection[i]
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">
Le <code>[i]</code> de Python ressemble plus au <code>.at(i)</code> de C++ : il vérifie explicitement que <code>i</code> n'est pas trop grand.
Les indices négatifs, quant à eux, comptent à rebours à partir de la fin.
</td></tr>
<tr>
<td>
{% highlight cpp %}
collection.back()
{% endhighlight %}
</td>
<td>
{% highlight python %}
collection[-1]
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
collection.empty()
{% endhighlight %}
</td>
<td>
{% highlight python %}
not collection
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Opérations de <em>modification</em> des vecteurs/listes</th></tr>
<tr>
<td>
{% highlight cpp %}
collection.clear(x)
{% endhighlight %}
</td>
<td>
{% highlight python %}
collection.clear(x)
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
collection.push_back(x)
{% endhighlight %}
</td>
<td>
{% highlight python %}
collection.append(x)
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
# ne renvoie rien
collection.pop_back()
{% endhighlight %}
</td>
<td>
{% highlight python %}
# renvoie la valeur qui
# a été supprimée
collection.pop()
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

## Structures de contrôles

Tout comme le corps des fonctions, les structures de contrôles introduisent leur corps avec `:`.
Le corps doit être indenté (de 4 espaces par convention), et se termine par indentation.

<table>
<thead><tr><th>C++</th><th>Python</th></tr></thead>
<tbody>
<tr><th colspan="2">Branchement conditionnel</th></tr>
<tr>
<td>
{% highlight cpp %}
if (cond) {
  thenBranch()
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
if cond:
    thenBranch()
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
if (cond) {
  thenBranch()
} else {
  elseBranch()
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
if cond:
    thenBranch()
else:
    elseBranch()
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
if (cond) {
  thenBranch()
} else if (otherCond) {
  otherBranch()
} else {
  elseBranch()
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
if cond:
    thenBranch()
elif otherCond:
    otherBranch()
else:
    elseBranch()
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">
Remarquez le mot-clef <code>elif</code> utisé en Python si on veut enchaîner plusieurs conditions.
On ne peut pas utiliser <code>else if</code>, car <code>else</code> doit directement être suivi d'un <code>:</code> et d'un bloc indenté.
</td></tr>
<tr><th colspan="2">Itération sur une collection</th></tr>
<tr>
<td>
{% highlight cpp %}
for (auto elem : collection) {
  body()
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
for elem in collection:
    body()
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Itération par indices</th></tr>
<tr><td colspan="2">De <code>0</code> (inclus) à <code>n</code> (exclu)</td></tr>
<tr>
<td>
{% highlight cpp %}
for (size_t i = 0; i < n; i++) {
  body()
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
for i in range(n):
    body()
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">De <code>start</code> (inclus) à <code>end</code> (exclu)</td></tr>
<tr>
<td>
{% highlight cpp %}
for (size_t i = start; i < end; i++) {
  body()
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
for i in range(start, end):
    body()
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">De <code>start</code> (inclus) à <code>end</code> (exclu) par pas de <code>step</code></td></tr>
<tr>
<td>
{% highlight cpp %}
for (size_t i = start; i < end;
      i += step) {
  body()
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
for i in range(start, end, step):
    body()
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">
Les itérations ci-dessus sont <em>valides</em> avec <code>n == 0</code> ou <code>end == start</code>.
Elles n'effectuent alors aucune itération, en Python comme en C++.
</td></tr>
<tr><th colspan="2">Boucles de type "Tant que"</th></tr>
<tr>
<td>
{% highlight cpp %}
while (cond) {
  body()
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
while cond:
    body()
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">
Python n'a pas de boucle de type "Répéter ... Tant que".
Si vraiment nécessaire, vous pouvez obtenir le même effet avec une boucle <code>while</code>.
</td></tr>
<tr>
<td>
{% highlight cpp %}
do {
  body()
} while (cond);
{% endhighlight %}
</td>
<td>
{% highlight python %}
while True:
    body()
    if not cond:
        break
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Branchement multiple</th></tr>
<tr>
<td>
{% highlight cpp %}
switch (value) {
  case 3:
    codeFor3()
    break;
  case 5:
  case 7:
    codeFor5Or7()
    break;
  default:
    codeForEverythingElse()
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
match value:
    case 3:
        codeFor3()
    case 5 | 7:
        codeFor5Or7()
    case _:
        codeForEverythingElse()
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">
L'instruction <code>match</code> de Python est plus versatile que le <code>switch</code> de C++.
Si vous voulez voir ce que vous pouvez faire d'autre avec, <a href="https://docs.python.org/3/tutorial/controlflow.html#match-statements">consultez sa documentation</a>.
</td></tr>
</tbody>
</table>

## Structures de données

Au premier semestre, en C++, vous n'avez étudié que les `struct`s comme structures de données personnalisées.
Il existe aussi des `class`es, mais ce n'est pas au programme du premier semestre.

Python ne possède *que* des `class`es.
Cependant, une forme particulière de classe en Python correspond à la notion de `struct` de C++.
Il s'agit des *data classes* (classes de données).

Une structure telle que

```cpp
struct Point {
  int x;
  int y;
}
```

correspond en Python à la dataclass suivante :

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int
```

<table>
<thead><tr><th>C++</th><th>Python</th></tr></thead>
<tbody>
<tr><th colspan="2">Création d'une instance</th></tr>
<tr>
<td>
{% highlight cpp %}
Point p { 5, 6 };
{% endhighlight %}
</td>
<td>
{% highlight python %}
p = Point(5, 6) # type implicite
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Lecture et modification des champs</th></tr>
<tr>
<td>
{% highlight cpp %}
cout << p.x << ", ", p.y << endl;
p.x = 7;
cout << p.x << ", ", p.y << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(f"{p.x}, {p.y}")
p.x = 7
print(f"{p.x}, {p.y}")
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Affichage d'un point complet (typiquement pour déboguer)</th></tr>
<tr>
<td>
N/A
</td>
<td>
{% highlight python %}
print(p)
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

En C++, on peut spécifier individuellement les champs qui ne peuvent pas être modifiés.
En Python, il faut décider pour la dataclass en entier si elle est muable ou pas.

La version immuable de `Point` s'écrirait en C++ comme

```cpp
struct Point {
  const int x;
  const int y;
}
```

et en Python avec une dataclass *frozen* :

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: int
    y: int
```

Dès la deuxième semaine du cours, nous introduirons la notion plus générale de classe, qui est au coeur de la méthodologie orientée objet.

## Pointeurs et références

Attention, ici, ça pique !
C++ et Python sont en désaccord fondamental sur les notions de valeurs, références, et pointeurs.
Au delà de toutes leurs différences de surface que nous avons listées jusqu'ici, c'est sans doute ici ce qui fait leurs réelles spécificités.
Nous en avons déjà parlé lors du tout premier cours, mais je tiens quand même à exposer ici ces concepts.

### C++ et son modèle par valeurs

En C++, on manipule les données *par valeur*.
Lorsque l'on assigne une variable à une autre, c'est la *valeur* qui est transférée.
Concrètement, cela veut dire qu'on effectue toujours une copie.
Par exemple :

```cpp
Point p { 5, 7 };
Point q = p; // copie de la valeur
```

![Points p et q comme valeurs, 1](/assets/img/point-values-1.svg)

Une conséquence directe est que, lorsque l'on modifie `p`, le point `q` n'en est pas affecté.

```cpp
p.x = 11;
cout << q.x << endl; // toujours 5
```

![Points p et q comme valeurs, 2](/assets/img/point-values-2.svg)

Avec un système par valeurs, la "boîte" nommée par une variable contient directement une valeur.
Il est possible de modifier le contenu de cette boîte (comme avec `p.x = 11`), et même de remplacer complètement la boîte (`p = Point { 13, 15 }`).
Dans un cas comme dans l'autre, toute autre copie de cette valeur reste inchangée.

Si nous souhaitons que `p` et `q` restent "connectées", nous pouvons utilisers des *pointeurs* à la place.
Dans ce cas, les *variables* `p` et `q` sont toujours des boîtes, mais les *valeurs* dans ces boîtes ne sont que des *pointeurs* vers d'autre boîtes.
Copier `q = p` revient donc à copier la *valeur pointeur*, et non pas la valeur *pointée*.

```cpp
shared_ptr<Point> p = make_shared<Point>({ 5, 7 });
shared_ptr<Point> q = p; // copie de la *valeur pointeur*
```

![Points p et q comme pointeurs, 1](/assets/img/point-pointers-1.svg)

Remarquez que la boîte de type `Point` n'a pas de nom !
Ni `p`, ni `q`.
Les boîtes qui portent un nom sont des valeurs pointeur.

Dans cette situation, si l'on modifie le champ `x` de la boîte pointée par `p`, naturellement, cette modification est reflétée sur `q`.
En effet, *c'est la même boîte* !

```cpp
p->x = 11;
cout << q->x << endl; // toujours 11
```

![Points p et q comme pointeurs, 2](/assets/img/point-pointers-2.svg)

Dans ce modèle, il y a une question très subtile : que se passe-t-il lorsqu'on *remplace* la boîte dans son entièreté ?
Est-ce que les deux pointeurs sont toujours connectés ?
En C++, il y a en fait moyen d'obtenir chacun de ces comportements, en fonction de l'instruction exacte qu'on utilise !

Si on utilise
```cpp
*p = Point { 30, 20 };
```
alors on suit d'abord la flèche du pointeur (l'opérateur `*`), *puis* on écrase le contenu de la boîte.
On se retrouve alors avec la situation suivante, dans laquelle `p` et `q` sont en effet toujours connectés.

![Points p et q comme pointeurs, 3](/assets/img/point-pointers-3.svg)

Par contre, si l'on écrit
```cpp
p = make_share<Point>({ 15, 17 });
```
alors c'est *la valeur pointeur* que l'on écrase, et l'on met une nouvelle flèche à la place.
On écrase la boîte nommée `p`, et non pas la boîte *pointée*.
On est alors dans une toute nouvelle situation, où `p` n'est plus connecté à `q`.

![Points p et q comme pointeurs, 4](/assets/img/point-pointers-4.svg)

#### *Aparté* : les références `&` de C++

Les références de C++, déclarées avec `&`, sont un modèle assez unique qui donne *plusieurs noms* à la *même* boîte.
Encore une fois, notez bien la différence avec les pointeurs : les boîtes nommées `p` et `q` sont bien des boîtes différentes quand ce sont des pointeurs.
Elles contiennent toutes les deux des flèches pointant vers la même (troisième) boîte, mais ce sont des boîtes différentes.

Avec les références de C++, si on les définit comme

```cpp
Point p { 5, 7 };
Point& q = p; // création d'un alias par référence
```

obtient une situation telle que celle-ci :

![Points p et q comme références, 1](/assets/img/point-cpp-references-1.svg)

Dans cette situation, même `p = Point { 15, 17 }` ne peut pas déconnecter `p` de `q`.
En effet, cette fois, ces deux noms correspondent exactement à la même boîte.

Il ne faudra pas confondre ce modèle de "références C++" avec le modèle par références de Python.
Les références C++ sont assez uniques en leur genre.
Le modèle par références de Python est plus répandu : on le retrouve dans de nombreux autres langages, tels que Java, Scala, JavaScript, etc.

### Python et son modèle par références

Le modèle de Python utilise un seul concept : la *référence à un objet*.
Dans ce modèle, *toutes* les variables (et champs, éléments de listes, etc.) contiennent une flèche.
Cette flèche pointe sur un *objet*, ou *instance* d'une classe (on utilise "objet" ou "instance" de manière interchangeable).

Lorsqu'on effectue une assignation, c'est donc bien la flèche qui est copiée, comme avec le modèle de *pointeurs* de C++.
Toutes les autres opérations, notamment les accès aux champs, suivent la flèche pour travailler sur l'objet.

Rappel : nous travaillons avec la dataclass `Point` (sa version muable), définie comme

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int
```

Avec les initialisations suivantes :

```python
p = Point(5, 7)
q = p
```

on obtient la situation que l'on avait avec les pointeurs en C++ :

![Points p et q comme références à une instance, 1](/assets/img/point-pointers-1.svg)

La boîte intitulée `Point` est une *instance* de la classe `Point`.
Les deux variables `p` et `q` contiennent deux flèches qui pointent vers cette même instance.

Si l'on modifie un champ de `p`, nous modifions en fait l'instance pointée par `p`.
Donc, `q` est affectée :

```python
p.x = 11
print(q.x) # 11
```

![Points p et q comme références à une instance, 2](/assets/img/point-pointers-2.svg)

> Anecdote : si vous jouez à des MMORPGs, vous avez sans doute l'habitude d'entrer dans une *instance* d'un donjon.
> Il s'agit exactement du même concept !
> Vous partagez cette instance avec votre groupe, c'est-à-dire que toutes et tous les membres du groupe ont une flèche vers la même instance du donjon.
> D'autres groupes qui attaquent le même donjon que vous (la même *classe* de donjon), au même moment, font référence à d'autres instances.
> C'est ainsi que votre groupe peut vaincre ensemble les monstres du donjon, sans pour autant que les autres groupes n'en soient affectés.

En revanche, si l'on réassigne `p` avec une nouvelle instance de `Point`, nous changeons la flèche de `p`, qui est donc déconnecté de `q`.

```python
p = Point(15, 17)
print(q.x) # toujours 11
```

![Points p et q comme références à une instance, 3](/assets/img/point-pointers-5.svg)

On peut donc considérer que *toutes* les variables Python sont des pointeurs.
On appelle cela des *références*.

Remarquez que Python n'a aucun équivalent de l'instruction C++

```cpp
*p = Point(30, 20);
```

C'était la variante qui suivait d'abord la flèche, *puis* écrasait la boîte.
Cette variante n'existe pas en Python.
Soit on remplace la flèche elle-même, soit on accède aux *champs* de l'instance.
On ne peut jamais écraser la boîte d'une instance elle-même.

### Quid des types primitifs ?

Si vous lisez d'autres sources de documentation sur Python ou d'autres langages, il est probable que vous y trouviez des explications du type :

> Les *primitives* sont passées par valeur, et les *objets* sont passés par référence.

Cette distinction est, à mon avis, une complication inutile.

En fait, même les primitives sont des objets !
Si l'on définit

```python
x = 5
```

on le représente graphiquement comme ceci :

![Les primitives comme des objets, 1](/assets/img/primitives-as-objects-1.svg)

Puisque les objets `int`s sont immuables, même si plusieurs variables ont une référence vers le même objet `int`, on ne pourra jamais le détecter.
Aucun code ne peut modifier le contenu de l'instance, et donc ne peut affecter d'autres parties du code !

C'est donc un atout majeur d'avoir des instances immuables (`frozen=True` pour les dataclasses).
Sauf bonnes raisons, il faudrait toujours définir nos classes comme immuables.

Mais... si même les `int`s sont des références... n'avons-nous pas menti sur nos diagrammes précédents avec les `Point`s ?
Eh oui !
Si l'on veut être exact, on devrait représenter les `Point`s comme ceci :

![Les primitives comme des objets, 2](/assets/img/primitives-as-objects-2.svg)

Pour des valeurs *immuables* simples (et notamment les `int`, `float`, `bool` et `str`), nous ferons souvent la simplification graphique que nous avons faite plus haut.
En effet, puisque personne ne peut voir la différence, c'est acceptable.

## Exceptions

<table>
<thead><tr><th>C++</th><th>Python</th></tr></thead>
<tbody>
<tr><th colspan="2">Lancer une exception</th></tr>
<tr>
<td>
{% highlight cpp %}
// Étant donnée une struct Error
// que nous aurions définie
throw Error { init... };
{% endhighlight %}
</td>
<td>
{% highlight python %}
# Python a un type d'Exception de base
# que nous allons utiliser.
raise Exception("error message")
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">
Contrairement à C++, où l'on peut lancer "n'importe quoi" comme exception, en Python les exceptions doivent être des instances des types prévus à cet effet.
Python possède un certain nombre de types d'exceptions prédéfinis.
Consultez [la documentation](https://docs.python.org/3/library/exceptions.html#Exception) pour plus d'information.
<code>Exception</code> est le type de base de la plupart des exceptions.
</td></tr>
<tr><th colspan="2">Attraper une exception</th></tr>
<tr>
<td>
{% highlight cpp %}
try {
  body();
} catch (Error e) {
  handler();
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
try:
    body()
except Exception as e:
    handler()
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">
Il peut y avoir plusieurs blocks <code>except</code>, si l'on veut traiter différents types d'exceptions.
</td></tr>
<tr><th colspan="2">Relancer une exception</th></tr>
<tr>
<td>
{% highlight cpp %}
try {
  body();
} catch (Error e) {
  handler();
  throw;
}
{% endhighlight %}
</td>
<td>
{% highlight python %}
try:
    body()
except Exception as e:
    handler()
    raise
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Effectuer des actions de "nettoyage"</th></tr>
<tr><td colspan="2">
Parfois, nous avons besoin d'effectuer des actions de "nettoyage" (cleanup) après un bloc de code.
Et ce, que le bloc de code se soit bien terminé, ou qu'il ait déclenché une exception.
C'est peu utile en C++ pour des raisons qui dépassent le cadre de ces explications, mais c'est courant dans le autres langages.
Les instructions <code>try..finally</code> sont faites pour cela.
</td></tr>
<tr>
<td>
{% highlight cpp %}
// Comment on devrait l'écrire en C++
try {
  body();
} catch (...) { // tout attraper
  // cleanup, puis relancer
  cleanup();
  throw;
}
// cleanup en cas de succès
cleanup();
{% endhighlight %}
</td>
<td>
{% highlight python %}
try:
    body()
finally:
    cleanup()
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

## Entrées/Sorties

### Formattage de chaînes

En C++, on utilise des *modificateurs* sur les flots pour formatter des données.
En Python, comme dans beaucoup d'autres langages, les fonctions d'entrées/sorties ne gèrent pas elles-mêmes le formattage.
À la place, on peut construire des *chaînes formattées*, et on envoie ensuite des chaînes formattées sur les sorties.
Cela veut dire qu'on peut aussi utiliser les mêmes mécaniques pour obtenir des `str` formattées, sans nécessairement les envoyer sur un flot.

<table>
<thead><tr><th>C++</th><th>Python</th></tr></thead>
<tbody>
<tr>
<td>
{% highlight cpp %}
cout << setw(6)
     << foo << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(f"{foo:6}")
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
cout << setw(6)
     << setfill('*')
     << foo << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(f"{foo:*>6}")
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
cout << setw(6)
     << setfill('*')
     << left
     << foo << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(f"{foo:*<6}")
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
cout << hex
     << foo << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(f"{foo:x}")
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
cout << fixed
     << foo << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(f"{foo:f}")
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
cout << scientific
     << foo << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(f"{foo:e}")
{% endhighlight %}
</td>
</tr>
<tr>
<td>
{% highlight cpp %}
cout << setprecision(6)
     << foo << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(f"{foo:.6}")
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">
Et on peut bien sûr tout combiner...
</td></tr>
<tr>
<td>
{% highlight cpp %}
cout << scientific
     << setprecision(6)
     << setw(15)
     << setfill('*')
     << left
     << foo << endl;
{% endhighlight %}
</td>
<td>
{% highlight python %}
print(f"{foo:*<15.6e}")
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

### Fichiers

<table>
<thead><tr><th>C++</th><th>Python</th></tr></thead>
<tbody>
<tr><th colspan="2">Écriture dans un fichier</th></tr>
<tr>
<td>
{% highlight cpp %}
ofstream f("file.txt");
f << "hello" << endl;
f.close();
{% endhighlight %}
</td>
<td>
{% highlight python %}
# le "w" indique "write"
with open("file.txt", "w", encoding="utf-8", newline='') as f:
    f.write("hello\n") # notez le \n
# close() automatique à la sortie du `with`
{% endhighlight %}
</td>
</tr>
<tr><td colspan="2">
<p>L'argument <code>encoding="utf-8"</code> s'assure que l'on manipule toujours les fichiers avec l'encodage UTF-8.
Sans cet argument, l'encodage peut varier d'un système à l'autre, ce qui est très problématique.</p>

<p>L'argument <code>newline=''</code> empêche Python de transformer les nouvelles lignes <code>\n</code> lorsqu'il écrit dans le fichier.
Encore une fois, sans cet argument, les résultats peuvent varier d'un système à l'autre.</p>
</td></tr>
<tr><th colspan="2">Lecture depuis un fichier</th></tr>
<tr>
<td>
{% highlight cpp %}
ifstream f("file.txt");
string line;
f >> line;
f.close();
{% endhighlight %}
</td>
<td>
{% highlight python %}
# le "r" indique "read"
with open("file.txt", "r", encoding="utf-8", newline='') as f:
    line = f.readline()
# close() automatique à la sortie du `with`
{% endhighlight %}
</td>
</tr>
<tr><th colspan="2">Lecture depuis l'entrée standard</th></tr>
<tr>
<td>
{% highlight cpp %}
cout << "Nom : ";
string name;
cin >> name;
{% endhighlight %}
</td>
<td>
{% highlight python %}
name = input("Nom : ")
{% endhighlight %}
</td>
</tr>
</tbody>
</table>

### Gestion des erreurs

C++ utilise des méchanismes silencieux pour les erreurs.
Il faut explicitement tester `f.fail()` pour gérer ces erreurs.

En Python, toutes les erreurs de manipulations de fichiers, ou de lecture de nombres, déclenchent des *exceptions*.
Il s'agit donc de traiter adéquatement les exceptions, s'il y a lieu.
Par exemple, ici on tente de lire deux entiers depuis un fichier, et on vérifie qu'il n'y a pas d'erreur.

```python
try:
    with open("file.txt", "r", encoding="utf-8", newline='') as f:
        x = int(f.readline())
        y = int(f.readline())
except OSError as e:
    print("Le fichier n'a pas pu être lu :")
    print(e)
except ValueError as e:
    print("Une des lignes ne contenait pas un entier en base 10 :")
    print(e)
```
