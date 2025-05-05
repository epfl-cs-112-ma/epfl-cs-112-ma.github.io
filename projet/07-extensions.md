---
title: Performances et extensions
layout: default
use_mathjax: true
---

# Performances et extensions

Nous arrivons tout doucement à la fin du projet.
Cette dernière série d'instructions vous demande d'évaluer les performances de votre implémentation, et d'ajouter une ou deux extensions personnalisées.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Évaluation des performances

Pour votre rapport de design (fichier `DESIGN`), nous vous demandons de faire une (petite) analyse des performances de votre jeu.

On s'intéresse ici particulièrement à deux choses :

* le chargement d'un niveau ;
* le `on_update`.

Pour chacun de ces deux aspects, nous vous demandons :

* une analyse "grossière" de complexité algorithmique ;
* des mesures concrètes de performance de type [benchmarks](/tutoriels/performances.html#benchmarking)

### Complexité algorithmique

Il y a beaucoup de "tailles" de données qui peuvent varier dans un niveau :

* la taille de la carte en nombre de "cellules" ;
* le nombre de murs fixes ;
* le nombre d'ennemis ;
* le nombre total de sprites faisant partie de plateformes mouvantes ;
* la taille maximale d'une plateforme mouvante ;
* le nombre d'actions d'un interrupteur ;
* etc.

Il est évident que nous ne vous demandons pas d'évaluer tous ces facteurs.

Nous vous demandons de choisir *deux* de ces facteurs qui vous paraissent pertinents :

* un facteur pour lequel vous évaluez la complexité du chargement d'un niveau ;
* un autre pour lequel vous évaluez la complexité de chaque `on_update`.

Choisissez des facteurs qui donnent lieu à une analyse intéressante.
Le nombre de murs fixes devrait presque "trivialement" être $\Theta(n)$ pour le chargement et $\Theta(1)$ pour le `on_update`.
Ce n'est donc pas un choix très intéressant.

D'autre part, vos algorithmes sont devenus compliqués.
Faites une analyse "grossière".
Il n'est pas nécessaire de justifier chaque petit bout.
Discutez de ce qui vous paraît intéressant.

En particulier, ne justifiez pas chaque bout de code qui est $\Theta(1)$.
La seule raison de mentionner un $\Theta(1)$, c'est pour un usage judicieux de `set`, de `dict`, ou de collision avec un spatial hash.

Pour vous donner une idée, on attend une quinzaine de phrases et/ou formules par analyse.

### Benchmarks

Pour les benchmarks, construisez (programmatiquement) des cartes qui font varier les paramètres que vous avez choisis pour vos analyses de complexité algorithmique.
Mesurez le temps mis pour exécuter chaque partie lorsque ces paramètres varient.
Prenez des valeurs qui s'étalent sur plusieurs ordres de grandeurs : par exemple 1, 3, 5, 10, 50, 100, 500, 1000, 10 000.

Pour mesurer `on_update`, vous pouvez directement appeler `on_update` "manuellement" depuis votre benchmark.
N'utilisez pas `window.test()`, puisque celle-ci fait réellement s'écouler le temps horloge !

Rapportez vos mesures sur deux graphes.

Commentez.
En particulier, vos mesures sont-elles en adéquation avec votre analyse théorique ?

## Extensions

D'ici ce jeudi, pensez à 2 extensions personnelles que vous voudriez ajouter à votre jeu.

[Enregistrez vos idées d'extension sur Moodle](https://moodle.epfl.ch/mod/assign/view.php?id=1336009).

Lors de la séance de jeudi, je passerai chez chaque groupe pour discuter de la faisabilité de vos idées.
En fonction de la difficulté que j'estime pour chaque idée, vous devrez ensuite implémenter une de ces idées, ou les deux.

N'oubliez pas de décrire vos extensions dans votre `README`, afin que votre correcteur ou correctrice puisse les essayer.
