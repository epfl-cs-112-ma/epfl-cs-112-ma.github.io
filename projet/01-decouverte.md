---
title: Découverte d'Arcade
layout: default
use_mathjax: true
---

# Découverte d'Arcade

Pour cette première semaine, nous allons découvrir la bibliothèque Arcade et ses principaux concepts.
Cette première série de tâches est fortement guidée, et peut difficilement être décomposée en parallèle.
Si vous avez déjà votre binôme, je vous conseille de :

* Développer ensemble sur une machine jusqu'au point indiqué (ou en passant d'une machine à l'autre, mais pas en même temps).
* Changer de personne qui *écrit* effectivement le code à chaque section.
* L'autre personne peut *lire* plus en détail ce document.

Si vous n'avez *pas* encore de binôme, cette semaine est encore une période de grâce.
Il est parfaitement possible de la compléter individuellement, en attendant de trouver votre binôme.
Vous pourrez alors partir de l'état de l'un ou de l'autre de vos projets respectifs.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Création du projet, affichage d'un écran bleu

### Création

Commencez par [créer un nouveau projet](/tutoriels/quick-projet-setup.html) pour votre... projet.

### Installation de Arcade

**tl;dr**: `$ uv add arcade`

Nous allons avoir besoin de la bibliothèque [Arcade](https://api.arcade.academy/en/latest/).
Il nous faut donc l'installer *dans notre projet*.
Heureusement, avec uv, cette tâche va se résumer à une ligne.

Comme avec toute bibliothèque, nous devons identifier quel est son **nom de paquet** (*package*).
La plupart des bibliothèques documentent comment les installer avec `pip`, mais pas avec `uv`.
Cependant, c'est le même nom de paquet dans les deux cas.
Nous allons donc à la section [Using pip](https://api.arcade.academy/en/latest/get_started/install.html#using-pip) de la page d'installation d'Arcade, qui nous dit :

> `pip install arcade` (ne pas exécuter ceci avec uv !)

Le nom du paquet est donc `arcade`.
Pour l'installer avec uv, nous exécutons donc :

```
$ uv add arcade
Resolved 17 packages in 907ms
Prepared 8 packages in 3.12s
Installed 8 packages in 554ms
 + arcade==3.0.1
 + attrs==25.1.0
 + cffi==1.17.1
 + pillow==11.0.0
 + pycparser==2.22
 + pyglet==2.1.2
 + pymunk==6.9.0
 + pytiled-parser==2.2.9
```

Ça y est ! Arcade est installée.

Nous pouvons remarquer que uv a installé une série d'autres bibliothèques.
C'est parce qu'Arcade dépend elle-même de celles-ci.
uv installe automatiquement toutes nos dépendences *transitives*.
Par ailleurs, elles sont isolées dans ce projet.
Elles ne viennent pas encombrer d'autres projets sur votre ordinateur (voire entre en conflit avec *leurs* dépendences respectives).

### Ouvrir une fenêtre vide

Il va nous falloir un peu de "boilerplate" pour démarrer notre jeu.
Le boilerplate, c'est le code pas très intéressant qui fait "tourner la boutique".

Commençons par donner le code, puis nous allons expliquer ses concepts :

```python
import arcade

# Constants
WINDOW_WIDTH = 1280
WINDOW_HEIGHT = 720
WINDOW_TITLE = "Platformer"

class GameView(arcade.View):
    """Main in-game view."""

    def __init__(self) -> None:
        # Magical incantion: initialize the Arcade view
        super().__init__()

        # Choose a nice comfy background color
        self.background_color = arcade.csscolor.CORNFLOWER_BLUE

        # Setup our game
        self.setup()

    def setup(self) -> None:
        """Set up the game here."""
        pass

    def on_draw(self) -> None:
        """Render the screen."""
        self.clear() # always start with self.clear()

def main() -> None:
    """Main function."""

    # Create the (unique) Window, setup our GameView, and launch
    window = arcade.Window(WINDOW_WIDTH, WINDOW_HEIGHT, WINDOW_TITLE)
    game_view = GameView()
    window.show_view(game_view)
    arcade.run()

if __name__ == "__main__":
    main()
```

Vous pouvez -- exceptionnellement -- copier-coller ce code directement dans votre `main.py`.
Vous pouvez immédiatement vérifier que tout se passe bien avec `uv run main.py`.

Détaillons maintenant ce que nous avons là, petit bout par petit bout.

On importe le module `arcade` que nous fournit la bibliothèque :

```python
import arcade
```

On définit quelques constantes pour leur donner des noms explicites.

```python
# Constants
WINDOW_WIDTH = 1280 # largeur de la fenêtre en pixels
WINDOW_HEIGHT = 720 # hauteur de la fenêtre en pixels
WINDOW_TITLE = "Platformer" # titre de la fenêtre
```

On définit notre classe principale, `GameView`.
On utilise une syntaxe particulière que nous n'avons pas encore vue, avec un qualificatif `(arcade.View)`.
Pour celles et ceux qui ont plus d'expérience : c'est la syntaxe qui nous permet *d'hériter de/étendre* `arcade.View`.
Pour les autres, acceptez pour l'instant que cette syntaxe est nécessaire pour que Arcade reconnaisse que notre classe représente notre jeu.

```python
class GameView(arcade.View):
    """Main in-game view."""
```

On définit ensuite le constructeur de notre classe.
Là aussi, une incantation magique, avec `super...`, se glisse dans le code.
Acceptez qu'elle est nécessaire en l'état, pour l'instant.

Après celle-ci, on initialise la couleur de fond d'écran.
Nous n'avons pas déclaré cet attribut.
C'est `arcade.View` qui l'a déclaré pour nous (oui, je sais, encore un peu de magie).

Finalement, on appelle notre méthode `setup()`.

```python
    def __init__(self) -> None:
        # Magical incantion: initialize the Arcade view
        super().__init__()

        # Choose a nice comfy background color
        self.background_color = arcade.csscolor.CORNFLOWER_BLUE

        # Setup our game
        self.setup()
```

La définition de cette méthode `setup()` ne fait rien pour l'instant.
Dans les sections suivantes, c'est là où nous installerons les différents éléments du jeu.

```python
    def setup(self) -> None:
        """Set up the game here."""
        pass
```

Finalement, nous définissons la méthode `on_draw()`, qui est responsable de dessiner l'état du jeu à un instant donné.

```python
    def on_draw(self) -> None:
        """Render the screen."""
        self.clear() # always start with self.clear()
```

La méthode `clear()` nous est donnée par `arcade.View`.
Elle va remplir l'écran de la couleur `self.background_color`.

Notez que nous n'appelons pas `on_draw` nous-mêmes.
C'est Arcade qui va appeler cette méthode environ 60 fois pas secondes.

Il ne nous reste plus que la fonction `main()`.
Celle-ci crée une instance de la classe `arcade.Window`, qui représente la fenêtre de l'application.
Le constructeur attend les largeur, hauteur et titre de la fenêtre en paramètres.

Ensuite, nous créons une instance de notre classe `GameView`.
Nous passons cette instance en argument à la méthode `window.show_view()`, ce qui va installer notre vue "dans la fenêtre".
C'est grâce à cela que Arcade saura appeler notre méthode `on_draw` !

Il ne reste plus qu'à lancer le "jeu" avec `arcade.run()`.

```python
def main() -> None:
    """Main function."""

    # Create the (unique) Window, setup our GameView, and launch
    window = arcade.Window(WINDOW_WIDTH, WINDOW_HEIGHT, WINDOW_TITLE)
    game_view = GameView()
    window.show_view(game_view)
    arcade.run()

if __name__ == "__main__":
    main()
```

Notez que nous ne connaissons pas l'implémentation de `arcade.Window`, pourtant nous pouvons l'utiliser correctement.
C'est le pouvoir des *interfaces* : nous n'avons pas besoin de connaître tous les détails de `arcade.Window`.

Il peut cependant être déroutant de comprendre ce code sans avoir une *idée* de la structure de `arcade.Window`.
Je vous propose donc, *pour une fois* une vue *schématisée, imaginée et grandement simplifiée* de ce que fait sans doute `arcade.Window` (je ne le sais pas moi-même, en fait !) :

```python
class Window:
    view: GameView

    def __init__(self, width: int, height: int, title: str) -> None:
        ... # toutes sortes d'initialisations

    def show_view(self, view: GameView) -> None:
        self.view = view

def arcade_run() -> None:
    window.open() # open the window on the user's screen
    while True:
        window.view.on_draw()
        wait_for(1 / 60) # wait for 1/60 seconds
```

Je vous suggère de créer un commit à ce moment.
De manière générale, il serait judicieux d'en faire un à chaque fin de section.

## Dessiner des objets : les Sprites

(Changer de membre du binôme qui "a le contrôle" ici.)

Il est temps de dessiner des objets à l'écran : notre personnage, un peu d'herbe et quelques caisses.
Dans un jeu, les objets ont un aspect *graphique* (dessin à l'écran) ainsi qu'un aspect *physique* (interactions).
Ceci est modélisé par la notion de *sprite*.
Chaque sprite représente un objet (ou parfois un morceau d'un objet) qui existe dans le monde 2D du jeu.

### Extraire GameView dans un module séparé

Avant d'aller plus loin, faisons un peu de *refactoring*.
Nous allons devoir étendre pas mal les responsabilités de `GameView`, et nous devrons bientôt tester certains de ses comportements.
Cela n'est pas pratique si elle se trouve dans le fichier `main.py`.

Créez donc un nouveau fichier `gameview.py` et déplacez-y la définition de la classe `GameView`.
Vous devrez insérer au début du `main.py` restant l'import suivant :

```python
from gameview import GameView
```

### Joueuse

Commençons par dessiner notre joueuse (*player*).
J'utiliserai le terme "joueuse" en français car les graphismes que nous utiliserons représentent une femme.
Il faut bien alléger l'écriture à un moment.

Créons donc un sprite pour notre joueuse.
Nous allons souvent manipuler celui-ci, donc nous allons le stocker dans un attribut de `GameView`.
Nous l'instancions dans `setup()`.

À partir d'ici, je vous recommende de ne *pas* copier-coller les bouts de code.
Prenez la peine des les écrire vous-mêmes, afin d'entraîner votre mémoire par la mécanisation des opérations.
Vous pouvez bien sûr copier-coller les chaînes de caractères, surtout celles qui contiennent des chemins vers des ressources.

```python
class GameView(arcade.View):
    player_sprite: arcade.Sprite

    def setup(self) -> None:
        """Set up the game here."""
        self.player_sprite = arcade.Sprite(
            ":resources:images/animated_characters/female_adventurer/femaleAdventurer_idle.png",
            center_x=64,
            center_y=128
        )
```

Un sprite est toujours caractérisé par :

* une *texture* (une image, si vous préférez), et
* une *position* dans le monde.

Nous chargeons une des textures libres de droits (licence CC0) "livrée avec" Arcade.
Vous pouvez trouver la [liste complète des textures disponibles](https://api.arcade.academy/en/latest/api_docs/resources.html).
Elles viennent toutes à l'origine du site [kenney.nl](https://kenney.nl/), qui propose de nombreuses textures libres de droits pour créer des jeux vidéos.
C'est une mine d'or !

Il ne nous reste plus qu'à *dessiner* notre joueuse dans la méthode `on_draw` :

```python
    def on_draw(self) -> None:
        """Render the screen."""
        self.clear() # always start with self.clear()
        arcade.draw_sprite(self.player_sprite)
```

Si vous lancez le jeu à ce stade, vous devriez voir une image proche du coin inférieur gauche de l'écran.

### À propos du système de coordonnées

![Système de coordonnées de Arcade](/assets/img/projet/arcade-coordinate-system.svg){: class="floating-illustration-image"}

Arcade utilise un système de coordonnées correspondant à ce que vous connaissez des mathématiques.
Le point $(0, 0)$ est en bas à gauche de la fenêtre.
Les $x$ positifs vont vers la droite, et les $y$ positifs vont vers le haut.

C'est peut-être contre-intuitif si vous avez déjà programmé des systèmes graphiques dans d'autres environnements.
Souvent, en informatique, les $y$ positifs vont vers *le bas*.

Par contre, les *angles* de rotation sont traités par Arcade dans le sens des aiguilles d'une montre (*clockwise*).
Ceci est *l'opposé* de ce qu'on utilise généralement en mathématiques.

Gardez cela en tête lorsque vous vous battrez avec des calculs de coordonnées plus tard dans le projet.

### Collections de sprites avec `SpriteList`

Nous avons maintenant notre joueuse, mais il lui manque un monde dans lequel évoluer.
Nous allons ajouter un bon nombre de sprites pour représenter de l'herbe et des caisses.

Nous pourrions les stocker dans une `list[arcade.Sprite]` de Python, puis faire une boucle `for` pour appeler `arcade.draw_sprite` sur chacun d'eux.
Cependant, cette approche poserait de sérieux soucis de performances.
À chaque appel à `draw_sprite`, Arcade doit interagir avec le GPU (Graphical Processing Unit) de votre ordinateur.
Ce va-et-vient est coûteux.
Il est beaucoup mieux de rassembler un grand nombre de sprites et de les envoyer tous ensemble au GPU.

C'est pourquoi Arcade a son propre type de collection, `arcade.SpriteList[T]`.
On peut appeler `draw()` directement sur une `SpriteList`, et cela envoie toute la collection d'un coup au GPU.
En fait, Arcade ne sait *que* utiliser `SpriteList`.
Quand on appelle `arcade.draw_sprite`, il crée une `SpriteList` temporaire pour l'envoyer au GPU !

On va donc déjà stocker notre joueuse dans une `SpriteList` dédiée, et l'utiliser pour la dessiner.

> Ce qui suit est une vue de `git diff`.
> Les lignes qui commencent par `@@` indiquent qu'on "saute" une section de code qui ne change pas.
> Par exemple, la création de `self.player_sprite_list = arcade.SpriteList()` est bel et bien dans la méthode `setup()`, pas dans la méthode `__init__()`.
> Référez-vous au contexte donné.

```diff
@@ -4,6 +4,7 @@ class GameView(arcade.View):
     """Main in-game view."""

     player_sprite: arcade.Sprite
+    player_sprite_list: arcade.SpriteList[arcade.Sprite]

     def __init__(self) -> None:
         # Magical incantion: initialize the Arcade view
@@ -22,8 +23,10 @@ class GameView(arcade.View):
             center_x=64,
             center_y=128
         )
+        self.player_sprite_list = arcade.SpriteList()
+        self.player_sprite_list.append(self.player_sprite)

     def on_draw(self) -> None:
         """Render the screen."""
         self.clear() # always start with self.clear()
-        arcade.draw_sprite(self.player_sprite)
+        self.player_sprite_list.draw()
```

Maintenant, créons notre monde.
L'herbe, comme les caisses, seront des obstacles qui ne bougeront jamais dans le monde.
C'est ce qu'Arcade appelle des *walls*.

Il est temps de vous faire réllement travailler un peu, donc cette fois nous ne faisons plus que décrire les étapes à réaliser :

1. Créez une nouvelle `wall_list` qui sera aussi une `SpriteList`.
   Construisez celle-ci avec `arcade.SpriteList(use_spatial_hash=True)`.
   Le "spatial hashing" augmente les performances pour une `SpriteList` qui ne contient que des sprites qui ne bougent (presque) jamais.
2. Dans `setup()`, écrivez deux boucles pour placer les sprites suivants dans cette `wall_list` :
    1. Des sprites d'herbe aux positions $y = 32$ et $x \in \\{0, 64, 128, \ldots, (1280-64)\\}$.
       Utilisez la ressource `":resources:images/tiles/grassMid.png"`.
       Ajoutez aussi le paramètre `scale=0.5` dans le constructeur de `arcade.Sprite`.
    2. Des sprites de caisses aux positions $y = 96$ et $x \in \\{256, 512, 768\\}$.
       Utilisez la ressource `":resources:images/tiles/boxCrate_double.png"`, ainsi que `scale=0.5`.
3. Dans `on_draw()`, n'oubliez pas de dessiner `wall_list`.

Vous devriez obtenir le résultat suivant :

![La joueuse, de l'herbe et des caisses](/assets/img/projet/player-grass-boxes.png)

## Contrôle de la joueuse au clavier

(Changer de membre du binôme qui "a le contrôle" ici.)

La prochaine étape est de pouvoir déplacer notre joueuse.
Pour cela, il va falloir réagir aux appuis sur les touches du clavier.

De la même façon que Arcade appelle `on_draw` pour dessiner l'état du jeu, elle appelle aussi deux méthodes supplémentaires pour le clavier :

* `on_key_press` est appelée lorsqu'une touche est enfoncée ;
* `on_key_release` est appelée lorsqu'une touche est relâchée.

### Mouvement latéral

Par exemple, à l'appui sur la touche flèche droite (→), on voudra démarrer un mouvement vers la droite.
Et quand cette touche est relâchée, on veut arrêter le mouvement.

Cela pose la question du *mouvement* !
Comme vous le savez, le mouvement est une évolution de la position dans le temps, qui dépend de la vitesse.
À son tour, la vitesse évolue en dépendant de l'accélération.

Pour un mouvement latéral à vitesse constante, nous n'allons pas modéliser l'accélération.
Nous allons directement changer la vitesse en fonction des appuis sur les touches gauche et droite.
Il se trouve que les `Sprite`s d'Arcade possèdent déjà des attributs `change_x` et `change_y` qui représentent la vitesse, comme `center_x` et `center_y` représentent la position.
Nous allons donc en profiter.

Déclarons une constante pour la vitesse latérale de la joueuse :

```python
PLAYER_MOVEMENT_SPEED = 5
"""Lateral speed of the player, in pixels per frame."""
```

puis ajoutons les méthodes suivantes dans `GameView` :

```python
    def on_key_press(self, key: int, modifiers: int) -> None:
        """Called when the user presses a key on the keyboard."""
        match key:
            case arcade.key.RIGHT:
                # start moving to the right
                self.player_sprite.change_x = +PLAYER_MOVEMENT_SPEED
            case arcade.key.LEFT:
                # start moving to the left
                self.player_sprite.change_x = -PLAYER_MOVEMENT_SPEED

    def on_key_release(self, key: int, modifiers: int) -> None:
        """Called when the user releases a key on the keyboard."""
        match key:
            case arcade.key.RIGHT | arcade.key.LEFT:
                # stop lateral movement
                self.player_sprite.change_x = 0
```

Notez que nous avons défini `PLAYER_MOVEMENT_SPEED` en "pixels per frame".
Une *frame* est la période entre deux calculs de l'évolution du monde + affichage du nouveau monde.
C'est donc bien une *unité de temps*.
Le *pixel* étant une unité de distance, il est bien vrai que "pixels per frame" est une unité de vitesse.

Arcade suppose plus ou moins qu'il y aura toujours 60 frames par seconde (FPS), donc $1 \text{ frame} = 1/60 \text{ seconde}$, et

$$ 5 \text{ pixels par frame} = 5 \text{ pixels par ($1/60$ seconde)} = 5 \cdot 60 \text{ pixels par seconde} = 300 \text{ pixels par seconde} $$

Mais alors quand modifions-nous la *position* ?
Idéalement, nous voulons le faire une fois par frame.
C'est exactement ce à quoi sert la méthode `on_update`.
Arcade appelle `on_update` une fois par frame, avant d'appeler `on_draw`.

```python
    def on_update(self, delta_time: float) -> None:
        """Called once per frame, before drawing.

        This is where in-world time "advances", or "ticks".
        """
        self.player_sprite.center_x += self.player_sprite.change_x
```

Vous pouvez maintenant lancer le jeu et déplacer la joueuse latéralement avec les flèches gauche et droite du clavier.

> D'un point de vue mathématico-physique, nous venons d'implémenter (une version rudimentaire) d'un intégrateur numérique.
> Les ajouts de `change_x` à `center_x` par pas de temps (1/60 seconde) reviennent à intégrer numériquement `change_x` par rapport au temps qui passe.
> Les moteurs physiques plus évolués prennent en compte les forces pour calculer les accélérations, puis intègrent celles-ci pour obtenir les vitesses, et intègrent également celles-ci pour obtenir les positions.
> Les nouvelles positions et vitesses sont alors utilisées pour calculer les nouvelles forces, et on répète ce manège.

### Obstacles et mouvement vertical

Vous aurez remarqué que notre joueuse n'est pas du tout arrêtée par les caisses, qui sont pourtant sensées être des obstacles.
De plus, elle ne peut pas sauter, ce qui est gênant pour un jeu de plateformes.
En effet, nous n'avons absolument pas tenu compte de la gravité ni des collisions dans notre mouvement&#x202F;!

La gravité en tant que telle est facile à gérer.
Il suffit d'intégrer l'accélération (constante `g` en pixels par frame²) pour mettre à jour `change_y` avant de mettre à jour `position_y`.
En revanche, si on ne fait que ça, notre joueuse va tomber dans l'infini, puisqu'aucune force contraire ne la retiendra lorsqu'elle "posera les pieds" sur de l'herbe.

Gérer ces collisions est une autre paire de manches.
Heureusement, Arcade nous donne une fonctionnalité toute faite pour gérer ces aspects physiques.
Si ce n'était pas le cas, nous aurions arrêté le projet ici pour cette semaine, et nous aurions passé une semaine rien qu'à gérer la physique.
Utilisons donc un `arcade.PhysicsEnginePlatformer` pour gérer la physique de notre joueuse.

Il faut le créer dans notre méthode `setup()`, avec quelques paramètres : le sprite de la joueuse, une sprite list à considérer comme les *murs* (et/ou sols, caisses, etc., tout ce qui est solide et empêche la joueuse de passer), et une constante de gravité.
On peut ensuite utiliser sa propre méthode `update` à la place de notre calcul.

```diff
@@ -3,6 +3,9 @@ import arcade
 PLAYER_MOVEMENT_SPEED = 5
 """Lateral speed of the player, in pixels per frame."""

+PLAYER_GRAVITY = 1
+"""Gravity applied to the player, in pixels per frame²."""
+
 class GameView(arcade.View):
     """Main in-game view."""

@@ -10,6 +13,8 @@ class GameView(arcade.View):
     player_sprite_list: arcade.SpriteList[arcade.Sprite]
     wall_list: arcade.SpriteList[arcade.Sprite]

+    physics_engine: arcade.PhysicsEnginePlatformer
+
     def __init__(self) -> None:
         # Magical incantion: initialize the Arcade view
         super().__init__()
@@ -50,12 +55,18 @@ class GameView(arcade.View):
             )
             self.wall_list.append(wall)

+        self.physics_engine = arcade.PhysicsEnginePlatformer(
+            self.player_sprite,
+            walls=self.wall_list,
+            gravity_constant=PLAYER_GRAVITY
+        )
+
     def on_update(self, delta_time: float) -> None:
         """Called once per frame, before drawing.

         This where in-world time "advances", or "ticks".
         """
-        self.player_sprite.center_x += self.player_sprite.change_x
+        self.physics_engine.update()

     def on_draw(self) -> None:
         """Render the screen."""
```

La joueuse est désormais correctement bloquée par les caisses, mais elle ne peut toujours pas sauter au-dessus !
Pour cela, il faut donner une vitesse vers le haut lorsqu'on appuie sur la flèche du haut.
En revanche, on "n'arrêtera" pas le mouvement vertical quand on relâche.
C'est le moteur physique qui va appliquer la gravité pour diminuer progressivement la vitesse verticale, jusqu'à retomber sur un mur.
On modifie donc uniquement `on_key_press`.

```diff
 PLAYER_GRAVITY = 1
 """Gravity applied to the player, in pixels per frame²."""

+PLAYER_JUMP_SPEED = 18
+"""Instant vertical speed for jumping, in pixels per frame."""
+
 class GameView(arcade.View):
     """Main in-game view."""

@@ -83,6 +86,9 @@ class GameView(arcade.View):
             case arcade.key.LEFT:
                 # start moving to the left
                 self.player_sprite.change_x = -PLAYER_MOVEMENT_SPEED
+            case arcade.key.UP:
+                # jump by giving an initial vertical speed
+                self.player_sprite.change_y = PLAYER_JUMP_SPEED

     def on_key_release(self, key: int, modifiers: int) -> None:
         """Called when the user releases a key on the keyboard."""
```

Ça y est !
Nous avons les bases d'un véritable jeu de plateformes.

### Remise à zéro

Il est parfois utile, surtout pendant vos tests, de redémarrer le jeu depuis le départ, dans l'état initial.
C'est justement ce que fait la méthode `setup()` !
Ajoutez un moyen de redémarrer depuis zéro lors de l'appui sur la touche Escape du clavier.

Consultez [la documentation](https://api.arcade.academy/en/latest/api_docs/arcade.key.html) ou profitez de l'autocomplétion pour découvrir la bonne constante pour cette touche.

## Caméra

(Changer de membre du binôme qui "a le contrôle" ici.)

Notre joueuse peut maintenant se déplacer.
Mais si elle s'aventure hors des bornes de l'écran, nous ne pourrons pas la suivre.
Normalement, dans un jeu de plateformes, l'écran suit les déplacements de la joueuse.

Puisqu'on ne veut pas déplacer le monde entier à chaque frame, il est préférable d'introduire une *caméra*.
C'est la caméra qui va bouger, et qui va déterminer quelle portion du monde est affichée à l'écran.
Pour l'instant, nous allons nous contenter d'une caméra très simple qui est constamment centrée sur la joueuse.

Créons d'abord une `arcade.camera.Camera2D`, et utilisons-là pour dessiner tous les sprites de notre monde.

```diff
@@ -18,6 +18,8 @@ class GameView(arcade.View):

     physics_engine: arcade.PhysicsEnginePlatformer

+    camera: arcade.camera.Camera2D
+
     def __init__(self) -> None:
         # Magical incantion: initialize the Arcade view
         super().__init__()
@@ -64,6 +66,8 @@ class GameView(arcade.View):
             gravity_constant=PLAYER_GRAVITY
         )

+        self.camera = arcade.camera.Camera2D()
+
     def on_update(self, delta_time: float) -> None:
         """Called once per frame, before drawing.

@@ -74,8 +78,10 @@ class GameView(arcade.View):
     def on_draw(self) -> None:
         """Render the screen."""
         self.clear() # always start with self.clear()
-        self.wall_list.draw()
-        self.player_sprite_list.draw()
+
+        with self.camera.activate():
+          self.wall_list.draw()
+          self.player_sprite_list.draw()

     def on_key_press(self, key: int, modifiers: int) -> None:
         """Called when the user presses a key on the keyboard."""
```

Pour l'instant, rien ne se passe, car nous ne faisons pas bouger la caméra.
On veut faire cela dans la méthode `on_update`, après avoir appliqué la physique du jeu.

```diff
     def on_update(self, delta_time: float) -> None:
         """Called once per frame, before drawing.

         This where in-world time "advances", or "ticks".
         """
         self.physics_engine.update()

+        self.camera.position = self.player_sprite.position
```

Aïe ! Mypy rapporte une erreur de type sur cette ligne :

> Incompatible types in assignment (expression has type `"tuple[float | int, float | int]"`, variable has type `"Vec2"`)

Néanmoins, Python étant ce qu'il est, nous pouvons quand même exécuter le programme sans erreur, et la caméra suit désormais la joueuse !

Pourquoi mypy se plaint-il ?
Il s'avère que [c'est un vieux bug dans mypy](https://github.com/python/mypy/issues/3004) qui a été [très récemment corrigé](https://github.com/python/mypy/pull/18510).
*Trop* récemment.
Le fix ne fait pas encore partie d'une release officielle de mypy.
J'espérais qu'elle arriverait avant le début du semestre, mais c'est raté ; nous attendons toujours.
On va donc faire ce qu'il est interdit de faire à moins de faire face à un bug de mypy : nous allons lui dire d'ignorer cette ligne !

```python
# Waiting for a new version of mypy with https://github.com/python/mypy/pull/18510
self.camera.position = self.player_sprite.position # type: ignore
```

## Ramasser des pièces

(Changer de membre du binôme qui "a le contrôle" ici.)

Nous arrivons au dernier concept important à connaître avant de pouvoir vous lâcher dans la nature d'Arcade : la détection de collisions.
Certes, le moteur physique s'occupe des collisions entre la joueuse et les *murs*, dans le but de contrôler ses mouvements.
Mais nous devrons aussi gérer les collisions pour d'autres aspects du jeu, notamment pour ramasser des objets.
En effet, on ne veut pas empêcher la joueuse de se rendre sur un objet.
Au contraire, on souhaite qu'elle le fasse, pour le ramasser et lui remettre.

Pour cela, nous allons utiliser des pièces à collectionner.
Commencez par ajouter quelques pièces dans le monde :

1. Créez une nouvelle sprite list appelée `coin_list` (les pièces étant fixes, activez le "spatial hashing" comme pour les murs).
2. Remplissez-là avec des pièces, aux positions $y = 96$ et $x$ allant de $128$ à $1250$ par pas de $256$.
    1. Utilisez la ressource `":resources:images/items/coinGold.png"`, ainsi que `scale=0.5`.
3. N'oubliez pas de les dessiner.

Puisque c'est une sprite list différente des murs, le moteur physique permet à la joueuse de passer dessus.
Mais il faut encore les ramasser.

Pour cela, dans notre méthode `on_update`, nous allons tester si la joueuse est "sur" une pièce.
Si c'est le cas, nous allons la retirer du monde, ce qui donnera l'illusion que la joueuse l'a ramassée.
Plus tard, vous pourriez compter les pièces ramassées dans un score, par exemple.

Pour tester si la joueuse est sur une pièce, nous devons en fait tester si le sprite de la joueuse est en *collision* avec le sprite d'une des pièces.
C'est une opération fréquente, et coûteuse si on ne s'y prend pas bien.
Heureusement, Arcade nous fournit exactement ce dont nous avons besoin : la fonction

```python
arcade.check_for_collision_with_list(self.player_sprite, self.coin_list)
```

renvoie la liste de toutes les sprites dans `self.coin_list` qui sont en collision avec `self.player_sprite`.
En pratique, cette liste contiendra 0 ou 1 élément, mais pourquoi ne pas la gérer de manière générique ?

L'autre méthode dont avez besoin est

```python
some_sprite.remove_from_sprite_lists()
```

qui retire `some_sprite` de toutes les sprite lists dans lesquelles elle se trouve.
En pratique, dans notre cas, cela sert à la retirer de la liste `self.coin_list`.

Utilisez ces deux méthodes dans `on_update` pour permettre à la joueuse de ramasser les pièces qu'elle touche.

### Astuce : afficher les "hit boxes"

Il y a des algorithmes avancés derrière la fonction `check_for_collision_with_list`, qui sortent du cadre de ce cours.
Cependant, il est bon de savoir qu'ils se basent sur la notion de "hit box".
Idéalement, on voudrait que deux sprites entrent en collision si et seulement si il existe un pixel en commun dans leurs images.
En pratique, cela est trop coûteux.

Arcade calcule donc une "hit box" pour chaque sprite : un polygone, souvent un octogone irrégulier, qui circonscrit l'image tout en l'approchant le plus possible.
L'algorithme de collisions n'a plus alors qu'à tester si les octogones se superposent, ce qui est beaucoup plus efficace que des formes arbitraires.

Vous pouvez demander à Arcade de dessiner les hit boxes des sprites d'une sprite list avec

```python
some_sprite_list.draw_hit_boxes()
```

dans votre méthode `on_draw`.
Essayez !

## Tests

(Changer de membre du binôme qui "a le contrôle" ici.)

Les jeux vidéos sont, de notoriété publique, difficiles à tester.
En effet, la "sortie" du programme n'est autre qu'un flux vidéo, dont il faudrait en théorie tester précisément certains pixels.
C'est évidemment très fastidieux, et très fragile : de petits changements de traitements sans conséquence peuvent casser des tests pour rien.

On va donc plutôt choisir de tester certains *états internes* de notre jeu.
Par exemple, au lieu de tester que le pixel $(300, 200)$ est de la bonne couleur, on va tester que la position de la joueuse est bien (proche de) $(300, 200)$.
Pour cela, il faut avoir directement accès à cet état interne depuis nos tests.
Si vous vous demandiez pourquoi nous avons défini tous nos attributs de manière publique, c'est pour ça.

Voyons donc comment tester que notre joueuse est capable de ramasser des pièces.

### Préliminaires : `conftest.py`

Arcade a une limitation majeure : il ne peut y avoir qu'une seule `arcade.Window` dans le programme.
Il n'est même pas possible d'en "détruire" une pour pouvoir en créer une autre.
Il va donc falloir tricher dans nos tests pour que tous les tests réutilisent la même `Window`.
Heureusement, chaque test pourra créer sa propre `GameView`, donc ce ne sera pas trop gênant.
La partie délicate est de faire en sorte que les tests aient *accès* à une `Window` globale, qui sera "remise à zéro" avant chaque test.

Cela étant vraiment compliqué à faire, on vous le donne.
Téléchargez le fichier [`conftest.py`](./conftest.py) et sauvegardez-le dans votre projet sous `tests/conftest.py`.
Grâce à l'existence de ce fichier, vos tests pourront demander accès à la `Window` globale avec un paramètre, sous la forme :

```python
def test_something(window: arcade.Window) -> None:
    # use the window in the test
```

On ne vous demande pas de comprendre le contenu de ce fichier.
Je ne l'ai pas écrit moi-même.
J'ai récupéré la mécanique utilisée par la bibliothèque Arcade elle-même, et l'ai simplifiée un tout petit peu.

### Un test

Voici un test pour s'assurer que la joueuse peut ramasser des pièces :

```python
import arcade

from gameview import GameView

INITIAL_COIN_COUNT = 5

def test_collect_coins(window: arcade.Window) -> None:
    view = GameView()
    window.show_view(view)

    # Make sure we have the amount of coins we expect at the start
    assert len(view.coin_list) == INITIAL_COIN_COUNT

    # Start moving right
    view.on_key_press(arcade.key.RIGHT, 0)

    # Let the game run for 1 second
    window.test(60)

    # We should have collected the first coin
    assert len(view.coin_list) == INITIAL_COIN_COUNT - 1

    # Jump to get past the first crate
    view.on_key_press(arcade.key.UP, 0)
    view.on_key_release(arcade.key.UP, 0)

    # Let the game run for 1 more second
    window.test(60)

    # We should have collected the second coin
    assert len(view.coin_list) == INITIAL_COIN_COUNT - 2
```

La méthode `window.test(frames)` nous est offerte par Arcade pour faciliter l'écriture de tests.
Elle génère les appels à `on_update` et `on_draw` pendant le nombre de frames spécifié en argument.
D'une certaine manière, on peut considérer qu'elle *fait avancer le temps*.

Les tests qui auront besoin d'une `Window` vont suivre la formule suivante :

1. Créer une `view: GameView` (potentiellement avec des paramètres, dans le future).
2. Utiliser `window.show_view(view)` pour installer cette vue dans l'unique fenêtre de test.
3. Simuler un scénario avec un mélange de :
    * `window.test(frames)` pour faire avancer le temps.
    * Appels à `view.on_key_press`, `on_key_release` pour simuler les appuis de touches.
    * Assertions qui testent l'état interne attendu à certains moments dans le temps.

Vous pourrez prendre exemple sur le test ci-dessus pour toutes vos fonctionnalités supplémentaires, en commençant par les améliorations de la prochaine section.

## Améliorations

**À partir d'ici, vous pouvez travailler séparément.**

Nous vous proposons quelques améliorations à l'ergonomie de votre jeu, qui se basent sur les concepts que vous avez déjà en main.

Répartissez-vous le travail !

### Meilleure gestion du clavier

Avez-vous remarqué que votre joueuse s'arrête parfois alors que vous appuyez encore sur une touche gauche/droite ?
Cela peut se produire avec la séquence suivante :

1. Enfoncer la touche droite
2. Enfoncer la touche gauche
3. Relâcher la touche droite

La joueuse s'arrête, alors qu'elle devrait continuer vers la gauche.

**Identifiez** le problème et **corrigez**-le.

### Gestion plus fine de la caméra

Souvent dans un jeu de plateformes, la caméra ne reste pas exactement centrée sur la joueuse.
La joueuse peut se déplacer dans les quatre directions sans que la caméra ne bouge.
La caméra ne se met à bouger que lorsque la joueuse s'approche "trop près" des bords.

**Adaptez** la gestion de la caméra pour suivre ce nouveau modèle.
Essayez différentes valeurs de "marges", c'est-à-dire combien de pixels doivent être visibles autour de la joueuse au minimum.

### Sauts multiples

Avez-vous essayé de sauter alors que vous étiez déjà dans les airs ?
Le jeu vous laisse faire !
On se croirait dans [Flappy Bird](https://flappybird.io/).

**Identifiez** le problème et **corrigez**-le.

Consultez [la documentation de `PhysicsEnginePlatformer`](https://api.arcade.academy/en/latest/api_docs/api/physics_engines.html#arcade.PhysicsEnginePlatformer) pour voir si un attribut ou une méthode pourrait vous aider dans cette tâche.

### Bruitages

Un jeu sans bruitages, c'est un peu monotome.
**Ajoutez** des sons lors des sauts et lors de la collecte des pièces.

Vous aurez besoin des fonctions suivantes :

* [`arcade.load_sound`](https://api.arcade.academy/en/latest/api_docs/api/sound.html#arcade.load_sound) (en mode `streaming=False`, par défaut)
* [`arcade.play_sound`](https://api.arcade.academy/en/latest/api_docs/api/sound.html#arcade.play_sound) (en mode `loop=False`, par défaut)

Il y a [des sons disponibles ici](https://api.arcade.academy/en/latest/api_docs/resources.html#sounds).
Nous suggérons :

* `":resources:sounds/coin1.wav"` pour ramasser une pièce
* `":resources:sounds/jump1.wav"` pour sauter

Faites en sorte de ne charger (`load_sound`) chaque son qu'une seule fois, et non pas à chaque fois que vous voulez le jouer.

Vous pourriez aussi naviguer dans la documentation de Arcade pour voir s'il y a plus d'explications ou d'exemples sur l'utilisation des sons.

**Remarque :** il n'est pas possible de tester automatiquement les sons ; nous ne nous attendons donc pas à ce que vous le fassiez.

## Compléter le log

N'oubliez pas de compléter [votre fichier `LOG.md`](./#rendu).

Il n'y a pas de question pour `ANSWERS.md` cette semaine.
