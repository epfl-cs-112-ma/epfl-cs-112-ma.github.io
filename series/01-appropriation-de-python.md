---
title: "Série 1 : S'approprier Python"
layout: default
use_mathjax: true
---

# Série 1 : S'approprier Python

[Solutions](https://github.com/epfl-cs-112-ma/solutions-serie-01) (excepté le dernier exercice, qui sera complètement personnalisé)

Cette première série a deux objectifs :

* Récapituler les différentes notions du premier semestre.
* S'approprier Python, ses concepts, et comment les notions du premier semestre s'y adaptent.

Avant de commencer cette série, il est prévu que vous ayez suivi les deux tutoriels de prise en main :

* [Prise en main de git et GitHub](/tutoriels/git-github.html)
* [Prise en main de Python et VS Code](/tutoriels/prise-en-main.html)

De plus, si ce n'est pas encore fait, assurez-vous d'avoir au moins *parcouru* la référence [Python vs C++](/tutoriels/python-vs-cpp.html).
Celle-ci récapitule comment chacun des concepts vus au premier semestre en C++ se traduit en Python.
Il sera utile de la garder à portée de main.

Nous vous recommandons de créer [un nouveau projet Python](/tutoriels/prise-en-main.html#créer-un-projet-python) pour chaque série d'exercices.
Cela vous permettra d'isoler plus facilement le contenu des différentes semaines.

Cette série contient 3 exercices :

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Arithmétique rationnelle

Cet exercice était déjà proposé en fin du premier semestre.
Nous vous proposons ici de le redécouvrir avec Python, afin de mieux vous familiariser avec ce langage.

Le but de cet exercice est de coder les bases de l'arithmétique des nombres rationnels.

Un nombre rationnel est défini par deux entiers $p$ et $q$, tels que :

* $q > 0$, et
* $p$ et $q$ sont premiers entre eux.

$p$ représente le numérateur et $q$ le dénominateur.

Par exemple : $\frac{1}{2}$ : $p=1$, $q=2$ ; $\frac{-3}{5}$ : $p=-3$, $q=5$ ; $2$ : $p=2$, $q=1$.

Quelle type de données utiliser pour représenter les nombres rationnels ?

Écrivez un module (fichier) `rational.py`, contenant cette structure de donnée, nommée `Rational`.
Pour rappel, préférez les structures *immuables* autant que possible.

Définissez aussi une fonction `frac_to_str()` permettant de convertir un rationnel en chaîne de caractères.

Testez votre fonction ([écrivez des tests avec **pytest**](/tutoriels/prise-en-main.html#écrire-un-test)) avec, au moins, les 3 rationnels donnés en exemple ci-dessus.
Le résultat de `frac_to_str` pour deux-ci devrait être :

```
1/2
-3/5
2
```

On veut maintenant implémenter les quatre opérations élémentaires.
Pour chacune, définissez une fonction implémentant l'opération, et écrivez des tests pytest adéquats.

N'oubliez pas de vérifier avec **mypy** que votre programme n'a pas de problèmes de types.
VS Code devrait vous l'indiquer immédiatement, mais il est bon, de temps en temps, de forcer mypy à tout vérifier.

### Addition

```python
def add(r1: Rational, r2: Rational) -> Rational:
```

définie par

$$ \frac{p_1}{q_1} + \frac{p_2}{q_2} = \operatorname{reduce}\left(\frac{p_1 \cdot q_2 + p_2 \cdot q_1}{q_1 \cdot q_2}\right) $$

où $\operatorname{reduce}(p/q)$ trouve les nombres $p_0$ et $q_0$ tels que $p_0/q_0 = p/q$ et $p_0$ et $q_0$ vérifient la définition donnée plus haut (en particulier sont premiers entre eux).

Pour la fonction `reduce`, on utilisera le calcul de PGCD avec l'algorithme d'Euclide : étant donnés $a > b \geq 0$,

$$
\operatorname{pgcd}(a, b) = \left\{ \begin{array}{l}
  a \text{ si } b = 0 \\
  \operatorname{pgcd}(r, a) \text{ sinon, où } r \text{ est le reste de la division euclidienne de }a\text{ par }b \\
\end{array} \right.
$$

Exemples :

* $\frac{1}{2} + \frac{-3}{5} = \frac{-1}{10}$
* $2 + \frac{-3}{5} = \frac{7}{5}$
* $2 + 2 = 4$

Remarque : toute sophistication de la formule ci-dessus (en particulier si $q_1 = q_2$) peut évidemment être implémentée pour la fonction addition.

### Soustraction

```python
def subtract(r1: Rational, r2: Rational) -> Rational:
```

définie par

$$ \frac{p_1}{q_1} - \frac{p_2}{q_2} = \operatorname{reduce}\left(\frac{p_1 \cdot q_2 - p_2 \cdot q_1}{q_1 \cdot q_2}\right) $$

Exemples :

* $\frac{1}{2} - \frac{-3}{5} = \frac{11}{10}$
* $2 - \frac{-3}{5} = \frac{13}{5}$
* $\frac{-3}{5} - \frac{-3}{5} = 0$

### Multiplication

```python
def multiply(r1: Rational, r2: Rational) -> Rational:
```

définie par

$$ \frac{p_1}{q_1} \times \frac{p_2}{q_2} = \operatorname{reduce}\left(\frac{p_1 \cdot p_2}{q_1 \cdot q_2}\right) $$

Exemples :

* $\frac{1}{2} \times \frac{-3}{5} = \frac{-3}{10}$
* $2 \times \frac{-3}{5} = \frac{-6}{5}$
* $\frac{-3}{5} \times \frac{-3}{5} = \frac{9}{25}$

### Division

```python
def divide(r1: Rational, r2: Rational) -> Rational:
```

définie par

$$
\frac{p_1}{q_1} \mathbin{/} \frac{p_2}{q_2} = \left\{ \begin{array}{ll}
  \operatorname{reduce}\left(\frac{p_1 \cdot q_2}{q_1 \cdot p_2}\right) & \text{si } p_2 > 0 \\
  \operatorname{reduce}\left(\frac{(-p_1) \cdot q_2}{q_1 \cdot (-p_2)}\right) & \text{si } p_2 < 0 \\
  \text{non définie} & \text{si } p_2 = 0 \\
\end{array} \right.
$$

Comment représentez-vous la notion de "non définie" pour une fonction dans Python ?

Exemples :

* $\frac{1}{2} \mathbin{/} \frac{-3}{5} = \frac{-5}{6}$
* $2 \mathbin{/} \frac{-3}{5} = \frac{-10}{3}$
* $\frac{-3}{5} \mathbin{/} \frac{-3}{5} = 1$

## Dérivation formelle

Le but est ici de faire un programme capable de dériver formellement des expressions arithmétiques.

### Arbres binaires

On utilisera pour cela une structure d'arbre binaire.
Un arbre binaire est une structure de donnée définie de façon récursive par :

un arbre binaire est

* soit vide ;
* soit une valeur et deux sous-arbres binaires.

Par exemple, si l'on note un arbre entre parenthèses, en premier son sous-arbre gauche, puis sa valeur et enfin son sous-arbre droit, alors
```
(((a) + (b)) * ((c) - (d)))
```
est un arbre binaire : la valeur est `*` et les deux sous-arbres sont `((a) + (b))` et `((c) - (d))`.
On peut le représenter graphiquement comme

![Arbre binaire représentant une expression arithmétique](/assets/img/binary-tree-expression.svg)

De même, `(a)` est un arbre réduit à une valeur (c.-à-d. avec 2 sous-arbres vides).

Dans un fichier `derivatives.py`, définissez une structure de données pour représenter les arbres binaires.

Dans cette structure de données, la valeur sera représentée par un caractère.
En Python, ce sera donc une `str` de longueur 1.

Nous suggérons d'utiliser la valeur `None` de Python pour représenter un arbre vide.
Vous pouvez utiliser la notation `A | None` pour le type d'une variable qui peut soit être un `A`, soit être la valeur `None`.
Pour tester si une valeur `x` est la valeur `None`, Python recommande la syntaxe `x is None`.

> Vous pourriez obtenir un avertissement du type *"Tree" is not defined (Pylance)* (ou une `NameError` à l'exécution) sur l'usage de `Tree` comme type au sein de la dataclass `Tree`.
> Si c'est le cas, ajoutez la ligne suivante *au tout début* du fichier :
>
> ```python
> from __future__ import annotations
> ```

Question : pensez-vous que votre structure doivent être muable ou immuable ?
Dans le doute, commencez par la rendre immuable.
Vous verrez bien si cela fonctionne ou pas.

### Création d'arbres

Écrivez une fonction permettant de créer un arbre binaire à partir d'une valeur et de deux sous-arbres.
Vous pourrez également créer une fonction qui créée ce que l'on appelle une *feuille*, c'est-à-dire un arbre réduit à sa seule valeur, dont les 2 sous-arbres sont vides.

### Affichage

Écrivez une fonction permettant d'afficher un arbre binaire en format parenthésé comme illustré plus haut.

On affichera récursivement le sous-arbre de gauche, puis la valeur puis (récursivement) le sous-arbre de droite.
N'oubliez pas d'écrire des tests pour vos fonctions !

Passons maintenant à la représentation des expressions arithmétiques.

On utilisera pour cela les arbres binaires.
Nous ne considérerons ici que les opérateurs binaires `+` `-` `/` `*` et `^` (puissance).
Par exemple, $a + b$ sera représenté par l'arbre
```
((a) + (b))
```
où `'+'` est la valeur du premier arbre et `(a)` et `(b)` ses sous-arbres.

De même, l'expression arithmétique $(a + b) \times (c + d)$ sera représentée par l'arbre
```
(((a) + (b)) * ((c) + (d)))
```

Construire les arbres représentant les expressions suivantes (4 expressions) :

* $x + a$
* $(x + a) \times (x + b)$
* $((x \times x) \times x) + (a \times x)$
* $(x^a) \mathbin{/} ((x \times x) + (b \times x))$

Indication : commencez par créer les arbres binaires représentant les expressions $a$, $b$ et $x$ (seuls), puis ceux représentant les expressions $x+a$, $x+b$ et $x \times x$.

### Dérivation

Écrivez une fonction `derivative` qui prend un arbre binaire en argument, supposé être une représentation d'une expression arithmétique valide, et un caractère représentant la variable par rapport à laquelle il faut dériver.
Elle doit retourner l'arbre binaire correspondant à l'expression arithmétique dérivée.

On procédera pour cela de façon récursive en se rappelant les règles usuelles de dérivation :

$$
\newcommand{\ddx}[1]{\frac{\mathrm{d} #1}{\mathrm{d} x}}
\begin{align*}
\ddx{a}                 & = 0 \text{ où $a$ est une constante (ici : caractère différent de $x$)} \\
\ddx{x}                 & = 1 \\
\ddx{(f+g)}             & = \ddx{f} + \ddx{g} \\
\ddx{(f-g)}             & = \ddx{f} - \ddx{g} \\
\ddx{(f \cdot g)}       & = \ddx{f} \cdot g + f \cdot \ddx{g} \\
\ddx{(f \mathbin{/} g)} & = \left(\ddx{f} \cdot g - f \cdot \ddx{g} \right) \mathbin{/} (g \cdot g) \\
\ddx{(f^a)}             & = a \cdot \ddx{f} \cdot f^{a-1} \text{ (où $a$ ne dépend pas de $x$, ce que l'on supposera ici)} \\
\end{align*}
$$



Note importante : on ne veut pas l'arbre minimal de dérivation !
En clair, on ne cherchera pas à simplifier les expressions obtenues.

Testez votre fonction de dérivation sur les 4 expressions précédentes (au minimum).

## La sorcière et ses invocations

Vous en voulez plus ?
Pourquoi ne pas reprendre votre [projet du premier semestre](https://moodle.epfl.ch/mod/assign/view.php?id=1300384), et le réimplémenter en Python ?

Il y a quelques questions de conception à revoir, étant données [les différences de modèles de valeurs et références entre Python et C++](/tutoriels/python-vs-cpp.html#pointeurs-et-r%C3%A9f%C3%A9rences).

* Si vous aviez fait une différence entre `MonsterID` et `Monster`, cette distinction est-elle encore utile ?
* Quelles structures devraient être muables ? Lesquelles peuvent rester immuables ?
* Comment adaptez-vous les références `&` des signatures C++ dans un contexte Python ?
