---
title: Plateformes et interrupteurs
layout: default
use_mathjax: true
---

# Plateformes et interrupteurs

Cette semaine et la suivante, nous allons encore ajouter quelques fonctionnalités : des plateformes et des interrupteurs.
Les plateformes sont comme des murs, mais se déplacent selon un schéma prévu pour la map.
Les interrupteurs peuvent modifier certains aspects de la map lorsqu'ils sont touchés par une arme.

Répartissez-vous les tâches de manière à pouvoir travailler indépendemment et en parallèle.

Nous rappelons qu'il est toujours nécessaire [d'écrire des tests](./01-decouverte.html#tests) pour les tâches qui vous sont demandées.

**Ces instructions sont valables pour 3 semaines de travail.**
Il n'y aura pas de nouvelles instructions pour le projet en semaine 8 ni en semaine 9.
Étant donné que la semaine d'interruption des cours se situe entre les semaines 8 et 9, sur le calendrier il se sera écoulé *4* semaines lors des prochaines instructions.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Plateformes

Dans un jeu de plateformes, une *plateforme*, en tant que telle, est un mur qui bouge.
Souvent, il s'agit de morceaux qui flottent dans les airs, et se déplacent soit de gauche à droite, soit de haut en bas.

Vous allez devoir ajouter le support pour les plateformes dans votre jeu.

Arcade vous propose déjà des outils pour gérer le déplacement de plateformes, et leur effet sur la joueuse (si la joueuse est sur une plateform, et que la plateforme bouge, la joueuse est entraînée par la plateforme).
Le `PhysicsEnginePlatformer`, en plus d'un paramètre `walls`, accepte aussi un paramètre `platforms`.
Il va tenir compte du `change_x`/`change_y` des sprites dans les plateformes pour :

  * les faire bouger et changer de sens quand ils atteignent leurs `boundary_left`/`boundary_right`/`boundary_top`/`boundary_bottom` ;
  * faire bouger la joueuse si elle se trouve sur une plateforme en train de bouger.

Consultez [l'exemple "Moving platforms"](https://api.arcade.academy/en/latest/example_code/sprite_moving_platforms.html) d'Arcade pour voir comment l'utiliser.

Le comportement des plateformes, une fois qu'elles sont mises en place, est donc pratiquement gratuit pour vous.

Ce que Arcade ne gère pas, c'est la notion de "bloc" de sprites qui forment un tout cohérent.
Chaque sprite individuel est une plateforme, avec sa propre vitesse et ses propres limites de mouvement.

Dans nos fichiers de map, cependant, nous allons vouloir considérer des blocs de sprites.
Ces blocs forment un tout cohérent, qui doit bouger ensemble.

Concrètement, on représente une plateforme dans notre fichier de map de la même façon que des murs.
Mais, on va pouvoir placer des symboles `← ↑ → ↓` pour indiquer qu'un bloc contigu de sprites doit se déplacer.

### Mouvement horizontal

Voici un exemple d'une plateforme qui se déplace lattéralement (on omet l'en-tête de configuration) :

```
    ---→→→→→→→→
 S                E
======££££££££=====
```

Cette platforme est constituée de 3 morceaux d'herbe (`-`).
Au temps $t_0$, elle est à sa position initiale, telle qu'indiquée dans la map.
Elle a une vitesse constante de norme $v_p$ (une constante globale en pixels/frame) qui va vers la droite.
Elle doit se déplacer dans cette direction jusqu'à ce que le `-` le plus à droite se retrouve à la position de la `→` la plus à droite.
Autrement dit, l'endroit le plus à droite où elle peut arriver correspondrait à cette situation, si elle était décrite dans un fichier de map :

```
            ---
 S                E
======££££££££=====
```

Ensuite, elle reviendra vers la gauche jusqu'à sa position initiale, et continuera le cycle.
Encore une fois, `PhysicsEnginePlatformer` peut se charger de tout cela.
Ce que vous devez faire, c'est calculer où chacun des 3 sprites doit commencer et finir.

On peut aussi utiliser des flèches `←`.
Dans ce cas, la vitesse initiale est vers la gauche.
La map suivante crée une plateforme qui parcourt le même chemin, mais qui commence tout à droite au temps $t_0$.

```
    ←←←←←←←←---
 S                E
======££££££££=====
```

La position de départ est importante, car elle détermine la synchronisation entre plusieurs plateformes de la même map.
Par exemple, les deux plateformes ci-dessous se toucheront presque à un moment dans le temps (alors que si elles avaient été définies avec le même chemin mais des `→` pour les deux, elles ne se rapprocheraient jamais l'une de l'autre).

```
    ---→→→→→→→→ ←←←←←←←←---
 S                           E
======£££££££££££££££££££=====
```

On peut avoir à la fois des `→` et des `←`.
Dans ce cas, les limites sont définies par la `→` la plus à droite, et la `←` la plus à gauche.
La vitesse initiale est dans ce cas toujours vers la *droite*.

```
    ←←←←←←←←---→→→→→→→→
 S                       E
======£££££££££££££££=====
```

Encore une fois, la position et la direction initiale sont importantes pour pouvoir synchroniser à souhait différentes plateformes.

### Mouvement vertical

De la même façon, les flèches `↑` et `↓` définissent des plateformes qui se déplacent verticalement.
Quand il y a les deux, la vitesse initiale va vers *le haut*.

Il n'est pas permis de combiner déplacements horizontaux et verticaux pour la même plateforme.
(Vous pouvez l'implémenter si vous le voulez, mais ce n'est pas demandé.)

### Définition d'un bloc

Les plateformes ne sont pas toujours planes.
La situation suivante crée une unique plateforme, qui se déplace d'abord vers la droite, puis vers la gauche :

```
                  ---
              xxxx
              x  ---→→→→→→→→
    ←←←←←←←←---
 S                              E
======££££££££££££££££££££££=====
```

Les règles qui régissent ce qui constitue un unique *bloc* sont les suivantes.
Deux sprites font partie du même bloc si, *dans la situation initiale*, les conditions suivantes sont remplies :

* chaque sprite est un mur (`= - x`), de la lave (`£`), une pancarte de fin (`E`), ou un interrupteur (`^`) ; *et*
* ils sont dans des cases de la grille de la map qui se touchent, soit gauche-droite, soit haut-bas (mais pas en diagonale).

De plus, si les sprites $A$ et $B$ sont dans le même bloc, et que les sprites $B$ et $C$ sont dans le même bloc, alors les sprites $A$ et $C$ sont dans le même bloc.
C'est-à-dire que l'on complète la relation par transitivité.

Dans la situation ci-dessus, la forme suivante constitue donc un bloc :

```
  xxxx
  x  ---
---
```

En revanche, le petit morceau `---` en haut à droite ne fait pas partie du bloc, puisqu'il ne touche le gros bloc qu'en diagonale (ce qui ne compte pas).

Remarquez qu'après un peu de temps, la platforme va bouger vers la droite et donc toucher ce petit morceau.
Cela n'a pas d'importance.
Seule la situation initiale compte !

Notez aussi que la texture des sprites n'a aucun impact sur la définition de "se toucher".
Même si l'image ne touche pas le bord du "carré" alloué au sprite, on considère que le sprite peut toucher un autre pour former un bloc.
Par exemple, les 3 `-` l'un sur l'autre constituent un bloc, malgré que les images ne se touchent pas :

```
            -
            -→→→→→→→→
    ←←←←←←←←-
 S                        E
======££££££££££££££££=====
```

### Définition des flèches

Les flèches sont regroupées en *séries*.
Une série contient toutes les flèches *d'un unique type* qui sont alignées.
Par exemple, ci-dessous vous voyez 6 séries de flèches (1 avec des `→`, 1 avec des `←`, 2 avec des `↑` et 2 avec des `↓`) :

```
            ↑
            ↑
   →→→→←←←←↓
          ↓↓↑
          ↓ ↑
```

Une plateforme est affectée par une série de `→` si *exactement 1* de ses sprites est à *gauche* de la `→` à gauche.
Il en va de même pour les 3 autres flèches.

Si une plateforme est affectée par plusieurs séries qui vont dans le même sens, c'est une erreur.
Si une plateforme est affectée par une série verticale *et* une série horizontale, c'est une erreur (à moins que vous n'ajoutiez le support pour cela).

### Exemple compliqué

```
                       ↑
                       ↑
                       ↑
                       ↑
                       ↑
                       ↑
          =£££=       xxx
          -----=      x x
            *  =      xxx
         ←←←x→→=→→→→→→→→↓
             ^ =        ↓
            ----
 S                        E
===========================
```

Il y a 3 blocs qui vont bouger dans cet exemple :

* un `x` tout seul qui va à droite puis à gauche, avec un cycle de $2 \cdot 5$ périodes ;
* 8 `x` qui forment un carré, qui vont d'abord vers le haut puis vers le bas, avec un cycle de $2 \cdot 8$ périodes ;
* la forme compliquée avec des `=`, des `-`, des `£` et un `^`, qui va vers la droite (et revient jusqu'à son point de départ vers la gauche), avec un cycle de $2 \cdot 8$ périodes.

Remarques :

* La pièce `*` ne relie pas le `x` tout seul au gros bloc, puisque les `*` ne font pas partie de blocs (et elle ne bougera pas) ;
* Le `x` tout seul et le `^` ne se touchent pas puisqu'ils sont en diagonale l'un de l'autre dans la situation initiale, et ne se rassemblent donc pas en un bloc ;
* Le gros bloc n'est pas "affecté" par les deux `→→` entre lui et le `x` tout seul, puisque le `-` qui touche les `→→` n'est pas à leur gauche ;
* Le `x` tout seul va se désynchroniser du gros bloc puisqu'ils n'ont pas la même période (ils vont parfois se toucher mais ça ne les rassemblera pas) ;
* Le gros bloc et les 8 `x` resteront synchronisés, et comme les `x` partent vers le haut, ils ne vont jamais se chevaucher en pratique (bien que leurs chemins se superposent).

### Monstres

Les monstres sont indépendants des plateformes.

**Vous pouvez supposer qu'il n'y a pas de blob posé sur une plateforme qui doit bouger.**
De même, vous pouvez supposer qu'aucune plateforme qui bouge ne viendra obstruer le chemin d'un blob.

### Conseils

Commencez par "hard-coder" une plateforme avec 3 morceaux d'herbe qui bougent, sans charger cette information depuis la map.
Assurez-vous de comprendre quelles informations vous devez donner au physics engine pour qu'il fasse ce que vous voulez.
N'attaquez le chargement depuis la map qu'après avoir fait cela.

Réfléchissez à votre algorithme sur papier avant de l'implémenter en Python.
Il s'agit d'un algorithme qui va être compliqué ; structurez-le.
Il y a plusieurs manières de faire ; ne vous lancez pas forcément dans la première idée qui vous vient à l'esprit.

N'oubliez pas les différentes [structures de données](/tutoriels/structures-de-donnees.html) que vous avez vues.

Il peut être judicieux de réfléchir *en commun* à l'algorithme, même si une seule personne dans votre binôme *implémente* l'alogithme que vous avez imaginé.

## Interrupteurs

Les interrupteurs ouvrent et ferment des portails lorsqu'ils sont frappés par une arme (épée ou flèche).

Nous ajoutons deux nouveaux types de cases dans nos fichiers de map :

* un interrupteur `^` (*switch*)
* un portail `|` (*gate*)

L'interrupteur a deux états : "on" et "off".
Par défaut, chaque interrupteur est dans l'état "off".
Vous aurez donc besoin de deux images :

* `:resources:/images/tiles/leverLeft.png`
* `:resources:/images/tiles/leverRight.png`

De plus, un interrupteur peut être "enabled" (activé, par défaut) ou "disabled" (désactivé).
Cela ne se reflète cependant pas dans son apparance.

Pour le portail vous pourriez par exemple utiliser cette image :

* `:resources:/images/tiles/stoneCenter_rounded.png`

Le portail est un mur, au sens où il ne permet pas à la joueuse de passer.
Cependant, il a vocation à disparaître et réapparaître, par l'action des interrupteurs.
Lorsqu'un portail a disparu, la joueuse peut passer.

Les portails ne font *pas* partie des blocs de sprites formant des plateformes qui bougent.
Les interrupteurs par contre, oui, comme mentionné dans la section sur les plateformes.

Les interrupteurs n'interagissent pas avec la joueuse.
Elle peut passer dessus comme s'ils n'existaient pas.

Lorsqu'un interrupteur *activé* est touché par une arme (épée ou flèche), il change d'état, puis peut effectuer des actions :

* ouvrir et/ou fermer un ou des portails ;
* se désactiver.

Les flèches sont détruites quand elles touchent un interrupteur ou un portail fermé.

Un interrupteur *désactivé* ne peut plus changer d'état, et donc plus effectuer d'action.

La grande question est : comment décide-t-on des actions que doit faire chaque interrupteur ?
Il peut y avoir plusieurs interrupteurs et plusieurs portails, par exemple.
On pourrait aussi imaginer que les interrupteurs puissent faire d'autres actions dans le futur (ou dans vos extensions).

On va donc devoir étendre notre section de configuration.
Voici un exemple de map qui possède deux interrupteurs.
Un des interrupteurs manipule deux portails, et l'autre en manipule un troisième.

```yaml
width: 24
height: 8
switches:
  - x: 0
    y: 7
    # this switch is off by default (no 'state')
    # it opens the gate at position (6, 1), then disables itself
    # (that means we won't be able to switch it back off, so nothing to do there)
    switch_on:
      - action: open-gate
        x: 6
        y: 1
      - action: disable
  - x: 8
    y: 6
    state: on # this switch is on by default
    # when we turn it off, we open access to the coin,
    # but we close access to the exit
    switch_off:
      - action: open-gate
        x: 14
        y: 2
      - action: close-gate
        x: 18
        y: 5
    # when we turn it back on, we do the opposite
    switch_on:
      - action: close-gate
        x: 14
        y: 2
      - action: open-gate
        x: 18
        y: 5
gates:
  - x: 18
    y: 5
    state: open # this gate is open when the game starts
  # the other gates start closed
---
^     x           x
--    x ^         x
      x-----      |
   ---x           =
      x   -----   =
---   x   x   |  x=
 S    |  xx * x  x=  E
========================
---
```

Le format général dans lequel est écrite cette configuration est du [YAML](https://yaml.org/).
YAML est un format similaire à JSON (dont nous avons parlé [dans la série 8](/series/08-patmat-et-bibliotheques.html)) : il peut représenter des valeurs primitives (nombres, chaînes de caractères, booléens), des listes et des dictionnaires.
Cependant, là où JSON est optimisé pour les machines, YAML est plutôt prévu pour être *écrit* (et relu) par des humains, puis interprété par des machines.
Les lignes qui commencent par `#` sont des *commentaires* en YAML (comme en Python) ; ces lignes peuvent donc bien faire partie du fichier, tel que montré ci-dessus.

YAML n'est pas un format facile à lire, pour un programme.
On ne s'attend pas à ce que vous puissiez écrire vous-même un algorithme pour "parser" correctement du YAML.

Vous devrez donc utiliser une bibliothèque adéquate pour lire tout ça.
Par "chance", nos quelques configurations existantes (`width`, `height` et `next-map`) s'avèrent déjà être du YAML valide.
On peut donc les intégrer au nouveau système.

Conseil : commencez justement par adapter votre lecture de la configuration *existante* en tant que YAML, avant d'ajouter les interrupteurs et les portails.

Il existe plusieurs bibliothèques disponibles pour lire du YAML en Python.
À vous de faire preuve de recherche, d'esprit critique et d'expérimentation pour choisir celle qui vous convient le mieux.

## Compléter le log et les réponses

N'oubliez pas de compléter [votre fichier `LOG.md`](./#rendu).

Répondez aux questions suivantes dans votre fichier `ANSWERS.md` :

* Quel algorithme utilisez-vous pour identifier tous les blocs d'une plateformes, et leurs limites de déplacement ?
* Sur quelle structure travaille cet algorithme ? Quels sont les avantages et inconvénients de votre choix ?
* Quelle bibliothèque utilisez-vous pour lire les instructions des interrupteurs ?
  Dites en une ou deux phrases pourquoi vous avez choisi celle-là.
* Comment votre design général évolue-t-il pour tenir compte des interrupteurs et des portails ?
