---
title: Niveaux et épée
layout: default
use_mathjax: true
---

# Niveaux et épée

La semaine dernière, vous avez implémenté un système de chargement de maps.
Bien qu'il a -- j'espère -- été utile pour écrire vos tests, il est sous-exploité dans le cadre du vrai jeu.
Nous vous demandons cette semaine d'ajouter un système avec plusieurs niveaux (levels).
Arrivée à la fin d'un niveau, la joueuse sera transportée sur la map suivante.

De plus, nous avons été cruels avec notre pauvre joueuse.
On a placé des monstres sur sa route, sans lui donner les moyens de se défendre.
Vous allez donc lui donner un épée.

Répartissez-vous les tâches de manière à pouvoir travailler indépendemment et en parallèle.

Nous rappelons qu'il est toujours nécessaire [d'écrire des tests](./01-decouverte.html#tests) pour les tâches qui vous sont demandées.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Épée

### Nouveaux assets

Pour cette section, vous aurez besoin de nouveaux assets (images) qui ne sont pas distribués par défaut avec Arcade.
Nous vous proposons [le pack d'items voxel](/files/kenney-voxel-items-png.zip).
Dézippez son contenu dans un nouveau dossier `assets` de votre projet, de sorte que, depuis la racine du projet, le fichier `assets/kenney-voxel-items-png/sword_silver.png` existe.
Ces images proviennent du [Voxel Pack de kenney.nl](https://kenney.nl/assets/voxel-pack) et sont sous licence CC0, comme les images distribuées par défaut.

Pour charger la texture de l'épée, vous devrez donc donner ce chemin de fichier, sans le `:resources:` habituel.
Nous recommandons une `scale` de 0.7 fois la scale utilisée pour toutes les autres textures jusqu'à présent.

```python
arcade.Sprite(
    "assets/kenney-voxel-items-png/sword_silver.png",
    scale=0.5 * 0.7,
)
```

### Comportement de l'épée

Les règles de l'épée sont les suivantes :

1. Lorsqu'on enfonce le bouton gauche de la souris, la joueuse donne un coup d'épée.
   L'image de l'épée apparaît à l'écran dans la bonne position.
2. À ce moment (et à ce moment uniquement), si l'épée touche un monstre, celui-ci est tué.
3. Lorsqu'on relâche le bouton gauche de la souris, l'image de l'épée disparaît de l'écran.

La joueuse donne un coup d'épée dans la direction de la souris.
Le point d'origine ne doit pas forcément être le milieu de la joueuse.
Vous pouvez trouver un point de référence qui est visuellement "plaisant".

De même, vous pouvez essayer différentes distances entre l'épée et le point de référence de la joueuse.
L'épée doit toujours être à la même distance du point de référence ; cliquer plus loin ne doit pas permettre de donner un coup d'épée à distance.

L'épée doit être orientée dans la direction de frappe.
Par exemple, si la souris se trouve au-dessus de la joueuse, l'épée doit pointer vers le haut.
Vous devrez calculer le bon angle de *rotation* pour le sprite de l'épée, en plus de sa position.
Attention : l'image de l'épée, telle que dans le fichier `.png`, pointe vers le haut à droite, à 45° de l'horizontale dans le sens trigonométrique.
Tenez-en compte dans vos calculs.

L'épée ne "bouge" pas pendant une frappe.
Elle n'a pas d'animation.
Elle apparaît directement à sa position de frappe, et ne bouge pas jusqu'à dispaître.

Lorsqu'un monstre est tué, il est retiré du jeu.
Vous pourriez vouloir jouer un son à ce moment.

### Réagir aux événements souris avec Arcade

Tout comme `on_key_press` et `on_key_release`, Arcade nous donne les moyens de réagir aux événements souris avec `on_mouse_press` et `on_mouse_release`.
Ces méthodes, définies dans `arcade.View` et prévues pour être surchargées (redéfinies, overridées) dans votre classe, ont les signatures suivantes :

```python
def on_mouse_press(self, x: int, y: int, button: int, modifiers: int) -> None:
    ...

def on_mouse_release(self, x: int, y: int, button: int, modifiers: int) -> None:
    ...
```

Les coordonnées `x` et `y` sont exprimées en coordonnées *écran* (screen coordinates).
Vous devrez les convertir en coordonnées *monde* (world coordinates) avant de les comparer à la position de la joueuse.

Le paramètre `button` indique quel bouton de la souris est concerné par l'événement.
Pour l'épée, on s'intéresse au bouton gauche, soit `arcade.MOUSE_BUTTON_LEFT`.

Ignorez les `modifiers`.
Dans vos tests, utilisez `0` pour leur valeur.

### Liens utiles

* [Documentation de `arcade.Sprite`](https://api.arcade.academy/en/latest/api_docs/api/sprites.html#arcade.Sprite), notamment ses propriétés `angle` et `radians`.
* [Documentation de `arcade.camera.Camera2D`](https://api.arcade.academy/en/latest/api_docs/api/camera_2d.html), notamment ses méthodes `project` et `unproject`.
* [Documentation de `arcade.Vec2`](https://pyglet.readthedocs.io/en/latest/modules/math.html#pyglet.math.Vec2), qui est en fait un alias pour `pyglet.math.Vec2`.
* La fonction mathématique [`math.atan2`](https://docs.python.org/3/library/math.html#math.atan2) (et les autres fonctions de ce module).

## Score

Votre joueuse ramasse des pièces, mais pour l'instant, rien ne montre combien elle en a.
On veut ajouter un compteur de score sur notre jeu.

Le score doit toujours refléter le nombre de pièces qu'a ramassées la joueuse depuis le début du jeu.
Si elle perd, le compteur est remis à zéro.

On veut voir apparaître le score à un emplacement *fixe* de l'écran, par exemple, en haut à gauche.
Lorsque la caméra se déplace pour suivre la joueuse, ces éléments de "UI" (user interface) ne doivent pas se déplacer.

Le moyen le plus simple d'obtenir cet effet est de définir une *deuxième* caméra.
Vous ne déplacerez pas cette caméra ; elle restera à sa configuration d'origine, dont la position $(0, 0)$ est en bas à gauche de l'écran.
Dessinez les éléments de UI dans un second bloc `with ... activate():`, utilisant cette nouvelle caméra.

Pour dessiner du texte à l'écran, utilisez [`arcade.Text`](https://api.arcade.academy/en/latest/api_docs/api/text.html).

## Niveaux

Pour pouvoir passer de niveau en niveau, nous allons étendre les capacités de notre de fichier de map.
Nous ajoutons un nouveau type de sprite qui signale la fin du niveau, et un attribut optionnel dans la configuration pour savoir quelle est la map suivante.

La fin de map sera indiquée par le caractère `E` (end).
Nous suggérons l'image associée `":resources:/images/tiles/signExit.png"`.

Dans la configuration, en plus de `width` et `height`, il est maintenant possible (mais pas obligatoire), d'ajouter un attribut `next-map`.
Celui-ci indique le fichier de map suivante.
Voici un exemple :

```
width: 8
height: 2
next-map: map2.txt
---
S      E
========
```

Lorsque la joueuse atteint le sprite de fin d'une map, le jeu charge la map `map2.txt` située dans le dossier `maps/`.
Elle démarre alors au point `S` de cette nouvelle map.
Vous pourriez aussi vouloir jouer un son à ce moment.

La joueuse doit préserver son score (son nombre de pièces ramassées) lorsqu'elle change de niveau.

Si la joueuse perd, elle reprend au début du *premier* niveau (*pas* du niveau en cours).

### Démarrer directement à un niveau supérieur

Pour vos tests manuels (lorsque vous lancez `main.py`), il pourrait être utile d'accepter la map initiale en *argument de votre programme*.
Par exemple, vous voudrez peut-être lancer directement le niveau 2 avec

```bash
$ uv run main.py map2.txt
```

Vous pouvez utiliser [`sys.argv`](https://docs.python.org/3/tutorial/stdlib.html#command-line-arguments) pour récupérer les arguments en ligne de commande.

## Compléter le log et les réponses

N'oubliez pas de compléter [votre fichier `LOG.md`](./#rendu).

Répondez aux questions suivantes dans votre fichier `ANSWERS.md` :

* Quelles formules utilisez-vous exactement pour l'épée ?
  Comment passez-vous des coordonnées écran aux coordonnées monde ?
* Comment testez-vous l'épée ?
  Comment testez-vous que son orientation est importante pour déterminer si elle touche un monstre ?
* Comment transférez-vous le score de la joueuse d'un niveau à l'autre ?
* Où le remettez-vous à zéro ?
  Avez-vous du code dupliqué entre les cas où la joueuse perd parce qu'elle a touché un ou monstre ou de la lave ?
* Comment modélisez-vous la "`next-map`" ?
  Où la stockez-vous, et comment la traitez-vous quand la joueuse atteint le point `E` ?
* Que se passe-t-il si la joueuse atteint le `E` mais la carte n'a pas de `next-map` ?
