---
title: Maps et monstres
layout: default
use_mathjax: true
---

# Maps et monstres

Cette fois, le projet est lancé pour de bon.
Cette semaine, nous vous demandons d'ajouter deux fonctionnalités à votre jeu.
D'une part, au lieu de "hard-coder" un peu d'herbe et des caisses dans votre `setup()`, vous devrez *charger* la map (carte) depuis un fichier.
D'autre part, vous allez ajouter des monstres pour mettre des bâtons dans les roues de la joueuse : de petits blobs (*slimes*) qui se déplacent latéralement sur l'herbe.
En petit bonus, on ajoute de la "lave" : un terrain qui fait perdre la joueuse si elle tombe dedans.

À ce stade-ci du projet, nous espérons fortement que vous avez trouvé votre binôme.
Répartissez-vous les tâches de manière à pouvoir travailler indépendemment et en parallèle.

Nous rappelons qu'il est toujours nécessaire [d'écrire des tests](./01-decouverte.html#tests) pour les tâches qui vous sont demandées.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Charger la map depuis un fichier

Jusqu'à présent, votre code de création de la map est "hard-codé".
C'est du code Python qui crée les sprites pour l'herbe, les caisses et les pièces, et les place à des coordonnées fixes.
C'est déjà fastidieux pour une bande d'herbe et quelques objets.
Imaginez ce que ce sera si vous devez concevoir une carte plus grande par vous-mêmes.

Il faut donc que nous puissions *charger* une map *depuis un fichier*.
De cette manière, nous pourrons éditer la map séparément du code.

### Format du fichier

Nous utiliserons un petit format de fichier simple, qui est plus ou moins visuel.
Cela vous permettra de concevoir votre propre map sans trop de difficultés, et sans avoir besoin de logiciel supplémentaire.
Voici un exemple du format que nous vous demandons de supporter :

```
width: 20
height: 6
---
*    -  -
---        -
     o*
    ---        -
 S          *o   x*
======£££££=========
---
```

Enregistrez cette map dans le fichier `maps/map1.txt`.

Le fichier représente une map en mode "grille".
On organise la map en cases carrées, avec (au plus) un sprite par case.

Le format se compose de deux sections.

La première est la configuration.
On doit y trouver au moins la taille de la map, en nombre de cases de la grille, que nous noterons $(w, h)$.
La taille est donnée par une `width` et une `height`.
Les lignes de configuration doivent suivre exactement le format `key: value` (une clef `key`, suivi des caractères `:` et espace, suivi de la valuer `value`).
La `width` et la `height` doivent être des entiers strictement positifs, exprimés en base 10.

La configuration s'achève avec exactement la ligne

```
---
```

Ensuite vient la carte à proprement parler.
Elle est composée de $h$ lignes.
Après les $h$ lignes vient à nouveau une ligne de terminaison `---`.

Chacune des $h$ lignes représente une ligne de la grille de la map.
La première ligne correspond à l'indice $h-1$ ; la dernière à l'indice $0$ (de sorte que visuellement les $y$ aillent vers le haut, comme dans Arcade).
Elle est constituée de $0$ à $w$ caractères (inclus) ; chaque caractère représentant une case de la grille (les indices vont de $0$ à $w-1$ de gauche à droite), et le sprite qui doit s'y trouver.
Le caractère espace (`' '`) indique qu'il n'y a pas de sprite à cet endroit.
La ligne peut contenir moins de $w$ caractères ; dans ce cas on suppose que les caractères "manquants" sont tous des espaces.
Une ligne peut être vide.
S'il y plus de $w$ caractères, c'est une erreur.

### Types de case

Les caractères ont la signification suivante :

| Caractère | Catégorie       | Asset suggéré                                  |
|-----------|-----------------|------------------------------------------------|
| `=`       | Wall            | `:resources:/images/tiles/grassMid.png`        |
| `-`       | Wall            | `:resources:/images/tiles/grassHalf_mid.png`   |
| `x`       | Wall            | `:resources:/images/tiles/boxCrate_double.png` |
| `*`       | Coin            | `:resources:/images/items/coinGold.png`        |
| `o`       | Monster (slime) | `:resources:/images/enemies/slimeBlue.png`     |
| `£`       | No-go (lava)    | `:resources:/images/tiles/lava.png`            |
| `S`       | Start           | le sprite de la joueuse                        |

Le `S` est un peu particulier.
Il représente la position de départ de la joueuse.
Il doit y en avoir exactement un sur la map.

La lave (`£`) et le monstre blob (slime, `o`) sont décrits dans les sections suivantes.
Les trois types de murs se comportent de la même façon.
Ils ont juste un visuel différent.
C'est ce qui est déjà le cas dans votre code de la première semaine pour l'herbe et les caisses.

Les `*` sont les pièces à collecter.

Tous les sprites doivent utiliser la même `scale`.
Nous recommandons `scala=0.5`.
C'est également le cas **pour la joueuse** (à changer, donc, par rapport à la semaine dernière).

### Votre tâche

Pour cet aspect de votre projet, vous devez :

* Lire et décoder un fichier de carte tel que décrit ci-dessus.
* Utiliser son contenu pour générer dynamiquement la map de votre jeu, au lieu de la créer manuellement.
* Si le fichier attendu ne peut pas être lu ou n'a pas le bon format, votre application doit émettre un message d'erreur à l'attention de l'utilisatrice ou l'utilisateur :
    * soit dans la console (avec `print()`), auquel cas le programme doit s'arrêter immédiatement (recommandé à ce stade de votre projet) ;
    * soit sous forme d'un écran dédié dans la fenêtre de jeu.

  Dans un cas comme dans l'autre, il n'est pas permis de laisser la "stack trace" (le "traceback") s'afficher à l'écran.

## Lave

La lave est un nouveau type de "terrain" à traiter dans votre jeu, que l'on peut qualifier de "no-go" ("n'y va pas").
Ses propriétés sont les suivantes :

* Elle n'empêche pas la joueuse de venir dessus.
* Lorsque la joueuse entre en collision avec de la lave, elle perd.

Quand la joueuse perd, elle recommence au début du jeu.
On peut jouer un son de "game over" à ce moment.
Par exemple, le son `:resources:sounds/gameover1.wav`.

### Votre tâche

Pour cet aspect de votre projet, vous devez implémenter ce nouveau type de terrain.
En particulier, ses aspects graphiques et son effet sur la joueuse.

## Monstres blobs

Les blobs (*slime* en anglais, traduction libre) sont de petits monstres qui peuvent uniquement se déplacer latéralement sur un sol plat.
Si la joueuse touche un blob, elle perd immédiatement (comme lorsqu'elle touche de la lave).

### Règles

Un blob doit toujours démarrer directement sur de l'herbe (soit `=` ou `-` en termes du format de fichier décrit plus haut).
Il se déplace à vitesse constante, vers la gauche ou vers la droite.
Il change de direction lorsque

* il arrive au bout de sa plateforme (soit parce qu'il y a du vide à côté, soit un autre type de sol) ; ou
* il touche un mur (catégorie wall).

Les images suivantes montrent précisément de quels cas de figure on parle.

![Limite de plateforme 1](/assets/img/projet/slime-platform-boundary-1.png)
![Limite de plateforme 2](/assets/img/projet/slime-platform-boundary-2.png)
![Limite par mur](/assets/img/projet/slime-touch-wall.png)

Rien d'autre n'affecte les blobs.
Il ne sont pas arrêtés par d'autres blobs, d'autres pièces, ou même la joueuse.
Deux blobs peuvent donc se superposer ou se croiser.

Remarque importante : on n'applique **pas** de "physique" aux blobs.
La gravité n'a aucun effet sur eux.
La raison pour laquelle ils ne tombent pas n'est pas qu'ils touchent du sol, mais bien qu'aucune gravité ne les affecte.
Si on ne les arrête pas au bord d'une plateforme, ils vont continuer à avancer tout droit, dans le vide, sans tomber.

Nous suggérons une vitesse de 1 pixel par frame pour le déplacement latéral des blobs.

Lorsque la joueuse touche un blob, elle perd immédiatement.

### Votre tâche

Pour cet aspect de votre projet, vous devez implémenter les blobs, avec les règles décrites ci-dessus.
En particulier :

* déplacement latéral ;
* détection des limites de la platforme ;
* détection d'une collision avec un mur ;
* effet sur la joueuse.

## Liens utiles

Surtout pour la lecture du fichier :

* [Méthodes de la classe `str`](https://docs.python.org/3/library/stdtypes.html#string-methods).
* [La fonction `int`](https://docs.python.org/3/library/functions.html#int) permet de convertir une chaîne de caractères représentant un nombre en base 10 en un `int`.
* [Lecture de fichiers, C++ vs Python](/tutoriels/python-vs-cpp.html#fichiers) (rappel)
* [Lecture de fichiers, documentation officielle](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files)
* [La fonction `textwrap.dedent`](https://docs.python.org/3/library/textwrap.html#textwrap.dedent) permet d'écrire des chaînes littérales sur plusieurs lignes dans le code source, tout en ayant une indentation propre.

Surtout pour les blobs :

* [Documentation de `arcade.Sprite`](https://api.arcade.academy/en/latest/api_docs/api/sprites.html#arcade.Sprite)
* Vous pourriez avoir envie d'utiliser les attributs `boundary_left` et `boundary_right` pour vos sprites de blobs ; leur valeur est ignorée par Arcade, donc vous en faites ce que vous voulez

## Compléter le log et les réponses

N'oubliez pas de compléter [votre fichier `LOG.md`](./#rendu).

Répondez aux questions suivantes dans votre fichier `ANSWERS.md` :

* Comment avez-vous conçu la lecture du fichier ?
  Comment l'avez-vous structurée de sorte à pouvoir la tester de manière efficace ?
* Comment avez-vous adapté vos tests existants au fait que la carte ne soit plus la même qu'au départ ?
  Est-ce que vos tests résisteront à d'autres changements dans le futur ?
  Si oui, pourquoi ?
  Si non, que pensez-vous faire plus tard ?
* Le code qui gère la lave ressemble-t-il plus à celui de l'herbe, des pièces, ou des blobs ?
  Expliquez votre réponse.
* Comment détectez-vous les conditions dans lesquelles les blobs doivent changer de direction ?
