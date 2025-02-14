---
title: "Série 1 : S'approprier Python"
layout: default
use_mathjax: true
---

# Série 1 : S'approprier Python

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

Testez votre fonction (écrivez des tests avec **pytest**) avec, au moins, les 3 rationnels donnés en exemple ci-dessus.
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
