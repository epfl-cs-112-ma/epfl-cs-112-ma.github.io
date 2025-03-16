---
title: Arc et chauves-souris
layout: default
use_mathjax: true
---

# Arc et chauves-souris

Cette semaine, nous complexifions deux aspects de notre jeu.
Nous ajoutons une nouvelle arme, l'arc à flèches, ainsi qu'un nouveau monstre, la chauve-souris.

Répartissez-vous les tâches de manière à pouvoir travailler indépendemment et en parallèle.

Nous rappelons qu'il est toujours nécessaire [d'écrire des tests](./01-decouverte.html#tests) pour les tâches qui vous sont demandées.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Arc à flèches

Pour l'instant, votre joueuse n'a qu'une épée à sa disposition.
Les clics gauches de souris permettent d'attaquer juste à côté de la joueuse.

Lorsque la joueuse clique avec le bouton *droit* de la souris, elle *change d'arme active*.
Par défaut, l'arme active est l'épée.
Lorsqu'elle clique avec le bouton *gauche* de la souris, elle attaque dans la direction de la souris *avec l'arme active*.

Les assets dont vous aurez besoin sont déjà dans [le pack d'items voxel](/files/kenney-voxel-items-png.zip) que vous avez ajouté la semaine dernière pour l'épée.

Ajoutez une icône en haut à gauche de l'écran qui montre en tous temps quelle est l'arme active.

Si l'arc à flèches est actif lors d'un clic de souris gauche, les événéments suivants doivent se produire :

1. L'image de l'arc apparaît (instantanément) dans la position de tir en direction de la souris.
   Contrairement à l'épée, l'arc lui-même ne fait rien aux monstres ; c'est juste une image.
2. En même temps, une flèche apparaît à la même position, dans la direction de tir.
   Elle a un vecteur vitesse initial dont la direction pointe vers la direction de tir.
   La *norme* de ce vecteur est une constante que vous choisirez.
3. Deux suites d'événements se passent alors indépendamment :
    * Lorsque la joueuse relâche le bouton gauche de la souris, l'arc disparaît.
      (Jusque là, il reste affiché sans bouger comme le fait l'épée.)
    * La flèche se déplace à chaque `on_update`.

Le comportement d'une flèche "en vol" est le suivant :

* Elle est affectée par une constante de gravité, que vous choisirez.
  Ce n'est pas forcément la même constante de gravité que pour la joueuse (on est dans un jeu vidéo).
* À chaque `on_update`, vous devez :
    1. Mettre à jour son vecteur vitesse à cause de la gravité, *puis*
    2. Mettre à jour sa position à cause de sa vitesse, *puis*
    3. Si elle a touché un monstre, le monstre est tué et la flèche est détruite.
    4. Si elle a touché un terrain quelconque (mur, lave, etc.), la flèche est détruite.
    5. Si elle tombe à une ordonnée inférieure à la hauteur de l'écran, la flèche est détruite.

Les flèches n'interagissent pas avec les autres sprites sur le jeu (comme les pièces ou la joueuse).
Elle passent à travers.

Il est possible de tirer plus de flèches alors qu'une flèche est encore en vol.
Chaque flèche est indépendante des autres.

**Important** : vous devez choisir des constantes de vitesse initiale et de gravité qui font en sorte que **la parabole de chute soit clairement visible à l'œuil**.

## Chauves-souris

Les chauves-souris sont un nouveau type de monstre, pour déjouer les plans de la joueuse.

Comme les blobs, si la joueuse touche une chauve-souris, elle perd.
De même, si l'épée ou une flèche touche une chauve-souris, celle-ci est tuée et retirée du jeu.

Vous aurez besoin d'un nouveau pack d'assets pour la chauve-souris : [le pack extended enemies](/files/kenney-extended-enemies-png.zip).
Le contenu de ce zip provient du [pack Platformer Art Extended Enemis de `kenney.nl`](https://kenney.nl/assets/platformer-art-extended-enemies), qui est aussi sous license CC0.

L'asset pour la chauve-souris sera donc `"assets/kenney-kenney-extended-enemies-png/bat.png"`.

On place une chauve-souri sur la carte avec un nouveau caractère : `v`.
Sa position de départ définit un "champ d'action" dans laquelle elle est autorisée à voler.
À vous de définir ce champ d'action : cela peut être un disque centré sur la position de départ, ou un rectangle, etc.
Le champ d'action de chaque chauve-souris est défini au chargement de la carte, et ne change plus après.

Le *comportement* de la chauve-souris est erratique et aléatoire.
Mais on ne veut pas non plus qu'elle soit *trop* aléatoire.
Il faut faire en sorte que son mouvement ait l'air plus ou moins naturel.

Voici d'abord quelques règles strictes à respecter :

* Les chauves-souris ne sont affectées par *aucun* type de sprite (à part les armes qui les tuent).
  Elles volent "à travers" les murs, la lave, les pièces, les autres monstres, etc.
* Elles ne sont pas affectée par une gravité.
* Elles ne peuvent pas sortir de leur champ d'action.
* Elles doivent avoir une norme de vitesse constante (mais dont la direction variera).
* Leur trajectoire doit avoir une part de hasard ; elle ne peut pas être entièrement déterministe.

Nous ne prescrivons pas de formule précise pour leurs mouvements.
Voici en revanche quelques "idées" qui peuvent vous lancer :

1. Commencez par la faire se déplacer à ligne droite, à vitesse constante.
   Si elle s'apprête à sortir de son champ d'action, changez de direction.
   Soit aléatoirement, soit en "rebondissant" sur la paroi invisible du champ d'action, soit en faisant demi-tour, etc.
2. Ensuite, toutes les "quelques" frames, donnez-lui l'opportunité de changer de direction *aléatoirement*.
3. Finalement, faites en sorte que cet "aléatoirement" ait plus de chances de rester plus ou moins dans la même direction qu'avant, jusqu'à avoir très peu de chances de faire un demi-tour complet.
   Autrement dit, votre tirage aléatoire ne doit plus être *uniforme*, mais être *biaisé*, d'une manière ou d'une autre, autour de la direction actuelle de la chauve-souris.

Pour générer des nombres aléatoires en Python, consultez la [documentation sur les distributions à valeurs réelles du module `random`](https://docs.python.org/3/library/random.html#real-valued-distributions).

## Compléter le log et les réponses

N'oubliez pas de compléter [votre fichier `LOG.md`](./#rendu).

Répondez aux questions suivantes dans votre fichier `ANSWERS.md` :

* Quelles formules utilisez-vous exactement pour l'arc et les flèches ?
* Quelles formules utilisez-vous exactement pour le déplacement des chauves-souris (champ d'action, changements de direction, etc.) ?
* Comment avez-vous structuré votre programme pour que les flèches puissent poursuivre leur vol ?
* Comment gérez-vous le fait que vous avez maintenant deux types de monstres, et deux types d'armes ?
  Comment faites-vous pour ne pas dupliquer du code entre ceux-ci ?
