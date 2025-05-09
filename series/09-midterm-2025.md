---
title: "Midterm 2025"
layout: default
use_mathjax: true
---

# Midterm 2025

[Solutions](https://github.com/epfl-cs-112-ma/solutions-midterm-2025)

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Instructions

Plus exactement, voici toutes les consignes officielles qui s'appliquaient à ce midterm :

* Vous disposez de une heure et quarante-cinq minutes pour faire cet examen (9h15 -- 11h00).
* Vous avez droit à toute documentation papier, mais aucun matériel électronique.
* Vous pouvez répondre en français ou en anglais.
* L'examen comporte une seule "situation problème".
  Lisez-la *complètement*, ainsi que toutes les questions, avant de commencer à répondre.
  Les questions peuvent guider votre processus de réflexion, mais il s'agit avant tout de résoudre la situation problème dans son ensemble.
* Lorsque vous écrivez du code, veillez à ce que les types soient corrects.
  Votre code devrait passer mypy en mode strict (même si vous ne pouvez pas le vérifier avec une machine), sauf subtilité que vous n'auriez aucun moyen de connaître.
* **Marquez l'indentation avec des lignes verticales.**
  L'indentation est importante en Python, mais a tendance à se perdre sur papier.
  Faites en sorte que vos fins de blocs soient claires.
* Si une classe doit comporter un `__init__` "standard" (avec des paramètres qui correspondent aux attributs, et où l'init ne fait qu'éventuellement appeler `super` puis assigner les attributs), vous pouvez simplement écrire "`... # init standard`" dans votre définition de classe.
* N'hésitez pas à "refactoriser" au long de votre examen en expliquant vos changements en français, ou en modifiant des réponses précédentes.
  Vous pouvez ajouter des méthodes, ajouter des classes, généraliser des fonctions, "déplacer" des choses, etc.
* Il n'est pas requis d'écrire d'`import`, ni de documentation, ni de `__str__`/`__repr__`, ni de tests.
* La *duplication* de code (y compris du "copier/coller") sera pénalisée (d'où le refactoring).
* L'usage de `cast`, `Any` ou `# type: ignore` sera pénalisé (si ça ne vous dit rien, tant mieux).

## Sujet : Fiefs de lapins

![Situation exemple](/assets/img/exams/midterm-2025-bunny-kingdom.png){: class="floating-illustration-image"}

Dans le jeu de société *Bunny Kingdom*, les joueurs et joueuses incarnent des factions de lapins.
Chaque faction contrôle des *fiefs* (anglais : *fiefdom*).
Les fiefs rapportent des points de victoire en fonction des *ressources* qu'ils produisent (anglais : *resource*), et des *tours de cités* qu'ils contiennent (anglais : *tower*).
Vous allez modéliser et implémenter les aspects nécessaires pour calculer les points produits par *un* fief.

Sauf mention contraire, les éléments du jeu ne changent pas au cours de la partie.

Il y a 5 types de ressources (anglais : *resource kind*) : bois, carotte, poisson, champignon et or (anglais : wood, carrot, fish, mushroom, gold).
Les différents types de ressources n'ont pas de pouvoirs spéciaux, mais ils sont importants car les points sont comptés en fonction de nombre de ressources *différentes* produites par un fief.

Un fief est une liste de *cases* (anglais : *square*).
Il y a 3 sortes de cases :

* Les *villages* (anglais : *town*), qui contiennent 1 tour de cité.
* Les *cases à ressources*, qui produisent 1 type de ressource fixé (parmi les 5 possibles).
* Les *cases vides* (anglais: *empty*), qui ne font rien de spécial.

Sur les cases à ressources et sur les cases vides (mais *pas* sur les villages), on peut construire *jusqu'à* 1 *bâtiment* (anglais : *building*).
La construction d'un bâtiment se fait pendant la partie ; une case peut donc commencer sans bâtiment et recevoir un bâtiment plus tard.
Il y a 3 sortes de bâtiments :

* Les *industries* (anglais : *factory*), qui produisent 1 type de ressource fixé (parmi les 5 possibles).
* Les *villes* (anglais : *city*), qui contiennent entre 1 et 3 tours de cité.
* Les *marchés* (anglais : *market*), qui vont pouvoir produire 1 type de ressource *au choix*.

Si une industrie ou un marché est construit sur une case à ressource, cette case peut (indirectement) produire deux types de ressources différents.
On peut aussi construire une ville sur une case à ressource, auquel cas, indirectement, elle produit une ressource *et* apporte sa ou ses tours au décompte.

Le calcul des points totaux d'un fief se fait comme ceci :

* On compte le nombre de types de ressources *différents* produits dans tout le fief ; par les cases à ressources, les industries et les marchés.
  Pour les marchés, on maximise le nombre de points totaux, donc on va choisir des ressources qui ne sont pas encore produites par d'autres choses dans le fief (même les marchés ne peuvent pas produire des ressources autres que les 5 disponibles).
* On compte le nombre total de *tours* dans le fief, en additionnant toutes les tours contenues dans les villes et les villages.
* On *mutliplie* ces deux valeurs.

Dans l'exemple en image, le fief produit 2 bois et 3 poissons (sur les cases), ainsi qu'un 1 or et 1 champignon (*via* des industries).
Il a 2 villages, avec chacun 1 tour, ainsi que deux villes, qui ont respectivement 2 et 3 tours.
On a donc un total de 4 ressources différentes et 7 tours.
Sans compter le marché, cela devrait donner 28 points.
Mais le fief contient aussi un marché, qui peut choisir de produire de la carotte.
On a ainsi 5 types de ressources différents, et donc 35 points.

## Questions

Quand vous définissez des classes, laissez de la place pour y ajouter des méthodes plus tard.
Vous pouvez toujours les ajouter à un autre endroit si vous n'avez plus de place.

**Remarque générale 1 :** à tout moment, vous pouvez décider de "revenir en arrière" pour ajouter des méthodes ou des attributs à vos classes, si vous le désirez.
Pensez à laisser de la place dans vos classes, au cas où.
Vous pouvez toujours aussi définir plus de classes, de méhodes et de fonctions que demandé, bien entendu !
Les points par question sont donc un peu flexibles.

**Remarque générale 2 :** quand on dit qu'une fonction doit *accepter* un certain type de données $B$, il est permis de l'écrire de sorte qu'elle accepte en fait des valeurs de type $A$ à la place, du moment que $B <: A$.

---

**1.** Définissez (en Python) un type de données pour représenter un *type de ressource* (resource kind).

---

**2.** Définissez (en Python et/ou sous forme de diagramme) un ou des types de données pour représenter un *bâtiment* (building).

---

**3.** Définissez et implémentez (en Python) une fonction `count_building_resources` qui accepte une `list` de bâtiments, et qui renvoie le *nombre* de ressources *différentes* produites par ces bâtiments (c'est donc un entier).
Ne vous préoccupez pas des cases dans cette question.

Commencez par définir votre fonction *sans tenir compte des marchés* (au moins sur votre brouillon).
Ensuite, adaptez-là pour tenir compte de ce que les marchés peuvent apporter.

---

**4.** Définissez et implémentez (en Python) une fonction `count_building_points` qui accepte une `list` de bâtiments, et qui compte le nombre de points totaux que produiraient ces bâtiments.
On peut imaginer qu'ils seraient tous construits dans un fief qui n'a que des cases vides.

---

**5.** Définissez (en Python et/ou sous forme de diagramme) un ou des types de données pour représenter une *case* (square).

---

**6.** Définissez et implémentez (en Python) une méthode `attempt_build` que l'on peut appeler sur une case, qui accepte en paramètre un bâtiment.
Cette méthode essaye de construire le bâtiment donné sur la case.
Elle doit échouer si la case est une case de village, ou si il y a déjà un bâtiment sur cette case.
Elle renvoie `True` si elle a réussi, et `False` sinon.
Cette fonction doit être la seule façon dont on peut construire un bâtiment sur une case.

---

**7.** Définissez et implémentez (en Python) une méthode `count_square_resources` qui accepte une `list` de cases, et qui renvoie le *nombre* de ressources *différentes* produites par ces cases (*uniquement* par les cases elles-mêmes ; *pas* par leurs bâtiments éventuels).

---

**8.** Définissez et implémentez (en Python) une méthode `square_buildings` qui accepte une `list` de cases, et qui renvoie une `list` de tous les bâtiments construits sur ces cases.

---

**9.** Finalement, définissez et implémentez (en Python) une méthode `total_points` qui accepte une `list` de cases (donc un fief), et qui renvoie le nombre total de points pour ce fief.
