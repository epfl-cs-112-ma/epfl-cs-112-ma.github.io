---
title: Performances
layout: default
use_mathjax: true
---

# Performances

Cette semaine, nous explorons des outils et techniques pour mesurer et améliorer les performances des programmes.
Au delà de l'aspect théorique de la *complexité temporelle*, que vous connaissez déjà, nous allons voir comment les *facteurs constants* influent sur nos programmes.
Nous allons donner des outils qui vous permettent de *mesurer* les performances, et ainsi pouvoir les *améliorer*.

On parle donc de performances *brutes* cette semaine (*raw performance* en anglais).
Elles complémentent les performances *théoriques* que sont la complexité temporelle.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Rappels : complexité temporelle

Jusqu'à présent --- dans ce cours comme en ICC ---, nous avons fait peu de cas des performances *brutes*.
Cela ne veut pas dire que nous avons ignoré les notions de performances en général.
En particulier, nous avons étudié la *complexité temporelle* des algorithmes.
Il y a quelques semaines, nous avons vu comment le choix de [structures de données](/tutoriels/structures-de-donnees.html) a un impact sur la complexité des opérations, et donc sur les performances de nos algorithmes.

La complexité temporelle est cependant une notion théorique, qui --- intentionnellement --- se détache des *constantes*.
Rappelez-vous que $\Theta(n) = \Theta(2n)$ ou même $\Theta(1000n)$.
Du point de vue de la complexité, un algorithme $A$ qui serait $k$ fois plus lent que $B$, où $k$ est une *constante* (indépendante de la taille des données) est en fait tout aussi bon que $B$.

Évidemment, la réalité n'est pas aussi simple.
Si un calcul a besoin de 1 seconde ou 1000 secondes, cela fait une réelle différence.

Dans votre projet, si votre `on_update` prend 10 ou 100 milliseconds, cela fait la différence entre un jeu fluide ou un jeu "qui lague".
En effet, pour obtenir un framerate de 60 frames par seconde, votre `on_update` a un *budget* de 16 ms.
Si vous dépassez ce budget, le jeu lague.

Alors pourquoi avoir tant insisté sur la complexité temporelle ?
Si elle ne nous informe pas beaucoup entre deux algorithmes de *même* complexité, elle reste extrêmement importante pour comparer des algorithmes de complexités *différentes*.
Entre un algorithme $A$ de complexité $\Theta(n^2)$ et un algorithme $B$ en $\Theta(n \log n)$, il vaudra toujours mieux utiliser $B$.

> Ce n'est pas *toujours* vrai.
> Dans certaines situations très spécifiques, qui sortent du cadre de cours, un algorithme de complexité théorique moins bonne peut gagner.
> Ce sera le cas seulement pour des données "suffisamment petites".
> Ces situations sont rares, et vous n'en rencontrerez probablement pas dans vos développements avant très longtemps.

## Mesurer, mesurer, mesurer

Le gros avantage d'une analyse de complexité, c'est qu'elle est *universelle* : elle est vraie pour toutes les machines, et pour tous les langages.
Les performances brutes, quant à elles, sont des mesures d'une implémentation particulière, sur une machine particulière, avec des conditions ~~de température et de pression~~ d'exécution particulières (comme d'autres programmes en train de tourner en même temps, par exemple un pic d'activité intempestif de votre antivirus ; ou littéralement la température de votre processeur).

Dès qu'on s'intéresse aux performances *brutes*, il y a 3 mots d'ordre : **mesurer, mesurer, et mesurer**.

Voici quelques *très mauvaises sources* d'évaluation des performances brutes :

* votre propre intuition ;
* ce qu'on a lu sur Internet, y compris *via* une IA ;
* ce que quelqu'un d'autre a mesuré, dans un autre contexte que le vôtre.

Voici quelques (seulement) *mauvaises* sources :

* votre propre connaissance approfondie de l'achitecture des ordinateurs, de l'assembleur, et du fonctionnement des machines virtuelles (telles que l'interpréteur Python) ;
* l'intuition et la connaissance approfondie *de votre enseignant* à propos des sujets ci-dessus ;
* des mesures faites en *isolation*, comme mesurer une petite boucle en dehors du contexte de votre programme réel.

La seule source d'informations adéquate, lorsqu'on parle de performances brutes, est de **mesurer** les implémentations **dans leur contexte réel**.

Alors de quels outils disposons-nous pour cela ?
Il en existe de deux sortes majeures, qui vont vous être utiles : le *benchmarking* et le *profiling*.

### Benchmarking

Le benchmarking (test de performance) consiste à mesurer le temps total que prend une sous-partie d'un programme.
Cela permet de comparer des implémentations alternatives, afin de garder la meilleure.
On note l'heure au début, l'heure à la fin, puis on fait la différence.
Mais même ça n'est pas si simple.

Comme les mesures de performances dépendent d'un grand nombre de facteurs extérieurs à votre code, mesurer *une* fois ne sert absolument à rien.
Peut-être que vous avez mesuré précisément au moment où votre antivirus s'est réveillé pour un pic d'activité ?
Ou peut-être qu'un courant d'air est venu raffraîchir temporairement votre CPU ?
Il est donc absolument critique de mesurer un grand nombre de fois la *même* opération, et de rapporter des *statistiques* sur les mesures.
(Trop) souvent, on se contente de la moyenne, mais on devrait aussi prendre en compte les écarts-type et les percentiles.

Pour faire ce type de mesures, Python propose l'outil [`timeit`](https://docs.python.org/3/library/timeit.html).
On peut l'utiliser en ligne de commande, mais souvent il est préférable de le manipuler directement depuis le code Python, grâce à son API.

La plupart des exemples que vous trouverez sur Internet passent des bouts de code sous forme de chaîne de caractères, comme ceci :

```python
import timeit
print(timeit.timeit("sum(range(0, 100))"))
```

Cependant, cette façon de faire ne permet pas d'accéder aux modules importés par votre programme, ni à des variables locales.
Il est plus flexible de passer une [lambda](./generiques-et-lambdas.html#lambdas) (qui ne prend aucun paramètre) :

```python
import timeit
print(timeit.timeit(lambda: sum(range(0, 100))))
```

Cette fonction affiche, sur ma machine, à l'instant où j'écris ces lignes, le résultat suivant :

```
0.7210329109802842
```

ce qui veut dire que `timeit` a mesuré 0.72 secondes pour notre petite boucle.
Est-ce que ça vous paraît un résultat réaliste ?

On l'a mentionné plus haut : mesurer *une* fois ne sert à rien.
Tellement à rien que `timeit` exécute le code qu'on lui donne 1 million de fois par défaut.
Le temps pris pour exécuter *une* fois la `sum` est donc plutôt de $0.72 \times 10^{-6}$ secondes, ou 0.72 µs.
C'est nettement plus raisonnable.
Rappelez-vous que les CPUs modernes tournent autour des 3 GHz, ce qui veut dire qu'ils peuvent exécuter de l'ordre de 3 milliards d'opérations élémentaires par seconde, ou encore 0.3 ns par opération élémentaire (à la *très* grosse louche ; on pense en termes d'ordres de grandeur).
0.72 µs = 720 ns, donc on serait autour des 2000 opérations élémentaires, ce qui paraît raisonnable pour additionner 100 nombres entiers.

Enfin raisonnable ... pour Python.
Normalement, on devrait être autour de quelques centaines maximum.
Mais Python étant ce qu'il est, on peut se réjouir de n'avoir "que" un facteur 5-10 fois plus lent que ce qui serait vraiment raisonnable.

Les benchmarks avec `timeit` peuvent être très utiles pour repérer les ordres de grandeurs, et pour *détecter* des problèmes de performances.
Si les résultats des mesures sont beaucoup trop élevées par rapport à ce qu'on attendrait, on sait qu'on va devoir corriger *quelque chose*, mais on n'a pas beaucoup d'idées sur *quoi* corriger.
Pour cela, on va se tourner vers le profiling.

#### Impact du spatial hash

Mais avant cela, je vous propose de faire un petit test dans votre projet.
On va voir très concrètement l'impact de `use_spatial_hash` sur les performances des détections de collision.

Créez un petit test avec une map qui contient 10 000 blocs de lave.
Quelque chose comme :

```
width: 20000
height: 2
---
  S
=====££££££££££££... 10 000 fois
---
```

Puis, dans un test, récupérez le sprite de votre joueuse (par exemple `player_sprite`) et la sprite list qui contient les éléments de lave (disons `lava_list`).
Et écrivez ce test `timeit` :

```python
number = 10000
res = timeit.timeit(lambda: arcade.check_for_collision_with_list(player_sprite, lava_list), number=number)
print(res / number)
```

> Si vous exécutez ceci *via* `pytest` (ce que j'ai fait car mon setup de test est plus facile à exploiter pour créer des maps personnalisées), vous devrez lui donner l'option `-s` pour qu'il n'avale pas les `print`.
> De plus, il est très utile ici de ne lancer qu'un seul test.
> En ligne de commande, on peut faire ceci :
>
> `$ uv run pytest -s tests/test_votre_test.py`

Notez qu'on passe ici explicitement `number=` pour dire à `timeit` combien de fois effectuer l'opération.
De plus, on divise le résultat par ce même `number` pour obtenir directement le temps par opération.
Chez moi, cela donne des mesures de l'ordre de 5 µs par opération.
Pas mal, pour tester les collisions avec 10 000 éléments de lave !
Comparez avec les 0.7 µs requises pour seulement additionner 100 nombres entiers.

Ce qui est encore plus intéressant, c'est de comparer les résultats quand il y a 10, 100, 1000, 10 000, 100 000 éléments de lave.
On peut faire un petit tableau des résultats :

| Éléments de lave | Temps par opération |
|-----------------:|---------------------|
| 10               | 4.677e-6 s          |
| 100              | 4.696e-6 s          |
| 1000             | 4.128e-6 s          |
| 10 000           | 4.369e-6 s          |
| 100 000          | 4.340e-6 s          |

Les résultats sont essentiellement constants !
Eh oui, puisque la lave est statique, sa sprite list était une bonne candidate à l'utilisation de `use_spatial_hash=True`.

Changeons cela en `use_spatial_hash=False`, et retentons l'expérience :

| Éléments de lave | Temps par opération |
|-----------------:|---------------------|
| 10               | 4.009e-6 s          |
| 100              | 2.664e-5 s          |
| 1000             | 2.668e-4 s          |
| 10 000           | 2.358e-4 s          |
| 100 000          | 1.193e-3 s          |

Cette fois, on sent passer les ordres de grandeurs !
On voit clairement une tendance *linéaire* se dessiner.

On a pu vérifier expérimentalement que la détection de collision *avec* spatial hash est $\Theta(1)$, alors que *sans* spatial hash, elle est $\Theta(n)$ avec $n$ le nombre d'éléments de la sprite list.

Il y a tout de même une anomalie : entre 1000 et 10 000, les mesures sont restées au même ordre de grandeur, de 2e-4 secondes.
C'est parce qu'autour de cette limite, Arcade change de stratégie d'implémentation.
Jusqu'à 1500 éléments dans la sprite list, Arcade utilise des calculs entièrement écrits en Python.
Au delà des 1500 éléments, elle emballe les données et les transfère au GPU, puis fait les calculs de collisions sur le GPU.

Le GPU peut alors exécuter les calculs beaucoup plus rapidements : environ 10 à 20 fois plus vite, apparemment.
Le coût de transfert au GPU est cependant important, et Arcade décide que ça n'en vaut la peine qu'à partir de 1500 éléments.
Pourquoi 1500 ?
Probablement parce que les développeurs ou développeuses qui ont écrit ce code ont *mesuré* que c'était la frontière où ça valait la peine.

On peut par exemple faire une régression linéaire *grossière* (il faudrait plus de données pour le faire bien) sur les données 100 et 1000 d'un côté, et 10 000 et 100 000 de l'autre.
On obtient alors deux droites qui montrent à quoi devraient ressembler les performances des deux approches (avec ou sans CPU) en fonction de la taille de la sprite list.
Pour les données ci-dessus, j'ai obtenu [ces deux graphes](https://www.wolframalpha.com/input?i=plot+2.66844%C3%9710%5E-7+x+%2B+-4.44444%C3%9710%5E-8%2C+1.06356%C3%9710%5E-8+x+%2B+0.000129444%2C+x%3D0%2C5000).
On voit que les lignes se croisent aux alentours de $x = 500$, ce qui a le mérite d'être du même ordre de grandeur que les 1500 choisis par Arcade.
Avec ces quelques données, on pourrait s'attendre à ce que choisir 500 comme limite entre les deux algorithmes serait idéal pour ma machine.
Évidemment il serait stupide de réellement prendre une décision sur base de si peu de données.
Mais si on utilise plus de mesures, ce type d'analyse permet de décider d'une valeur raisonnable pour passer d'un algorithme à l'autre.
On peut supposer que les analyses faites pour décider du 1500 d'Arcade ont été faites d'une façon similaire.

Notez que les 100 000 sont tout de même 10 fois plus lents que les 10 000.
Une fois sur le GPU, on a toujours une complexité linéaire !
On a "seulement" enlevé un facteur 10 *constant*.

> En faisant les mêmes tests sur vos machines, vous obtiendrez sans aucun doute des résultats différents.
> Ils pourraient même être 10 fois plus rapides, ou 10 fois plus lents.
> Cependant, les *ratios* entre les différentes valeurs devraient survivre (approximativement).
> Vous devriez observer des résultats essentiellement constants pour le spatial hash, et essentiellement linéaires sans le spatial hash.
> C'est encore une fois une illustration de l'universalité des *complexités*, alors que les performances *brutes* sont très dépendantes des circonstances.
> Vous devriez aussi observer la cassure entre 1000 et 10 000 dans le cas sans spatial hash, bien que ça ne soit pas garanti par un changement de complexité.

### Profiling

Les mesures d'ensemble avec `timeit` permettent de comparer des alternatives qui ont des différences de timing importantes.
Elles permettent aussi de détecter de réels *problèmes* de performances : quand un code prend trop longtemps par rapport à ce qu'il devrait faire.
Cependant, ces mesures sont pratiquement inutiles pour *identifier* où sont les problèmes, et donc identifier ce qu'il faut améliorer ou corriger.

Beaucoup de développeurs et développeuses se fient à leur intuition pour régler les problèmes de performance.
Elles *supposent* ou *devinent* une source des problèmes, et s'attaquent à optimiser cette source supposée.
Cependant, rendre une partie du code 10 fois plus rapide ne sert absolument à rien si elle n'était en fait responsable que de 5 % du temps total passé dans un programme.
Cela a moins d'impact que de rendre seulement 10 % plus rapide la partie du code qui prend les 95 autres pourcents du temps.

Il nous faut donc un outil qui nous dise précisément *où le programme passe le plus de son temps*.
Un fois que l'on a cette réponse, on sait où il vaut la peine de focaliser nos efforts d'optimisations.
Soit pour trouver et utiliser un algorithme de meilleure complexité (la meilleure option si elle existe), soit pour utiliser des techniques plus spécifiques pour réduire les facteurs constants.

L'outil qui va nous donner ces réponses est le *profiler* (profileur en français, mais on utilise peu ce terme).
Le profiling monitore l'exécution d'un programme, et détermine combien de temps est passé dans chaque fonction.
Il rapporte ensuite ces données, ce qui permet d'identifier les fonctions qui contribuent le plus au temps d'exécution du programme.

Python offre le module [`cProfile`](https://docs.python.org/3/library/profile.html).
Tout comme `timeit`, on peut l'utiliser en ligne de commande, mais il est plus flexible de l'utiliser *via* son API.
Illustrons son utilisation dans le cadre du projet, pour voir où `on_update` passe le plus clair de son temps.

```diff
diff --git a/gameview.py b/gameview.py
index 0eb0a84..4062fd1 100644
--- a/gameview.py
+++ b/gameview.py
@@ -1,8 +1,10 @@
 import arcade
+import cProfile
 ...

 TILE_SCALING = 0.5
@@ -270,6 +272,8 @@ class Bow(Weapon):
 class GameView(arcade.View):
+    profiler: cProfile.Profile
+
     ...

@@ -301,6 +305,8 @@ class GameView(arcade.View):
    def __init__(self) -> None:
         super().__init__()
+
+        self.profiler = cProfile.Profile()

         ...
@@ -440,10 +446,11 @@ class GameView(arcade.View):

     def on_update(self, delta_time: float) -> None:
+        self.profiler.enable()
+        self.do_on_update(delta_time)
+        self.profiler.disable()
+
+    def do_on_update(self, delta_time: float) -> None:
         self.physics_engine.update()

         ...
diff --git a/main.py b/main.py
index 17a5b74..481da6a 100644
--- a/main.py
+++ b/main.py
@@ -10,11 +10,16 @@ WINDOW_TITLE = "Platformer" # titre de la fenêtre
 def main() -> None:
     """Main function."""

     # Create the (unique) Window, setup our GameView, and launch
     window = arcade.Window(WINDOW_WIDTH, WINDOW_HEIGHT, WINDOW_TITLE)
     game_view = GameView()
     window.show_view(game_view)
     arcade.run()

+    game_view.profiler.print_stats(sort='cumtime')
+
 if __name__ == "__main__":
     main()
```

Dans le constructeur de `GameView`, on crée une nouvelle instance de `Profile`.
On sort le réel contenu de `on_update` dans une autre méthode `do_on_update`.
Dans `on_update`, on active le profiler, on exécute `do_on_update`, puis on désactive le profiler.
Avec Arcade, il sera nécessaire de profiler spécifiquement `on_update` et pas le reste, sinon on aura des problèmes pour les flame graphs (voir plus loin).
Finalement, à la fin du `main()`, on demande au profiler d'imprimer ses mesures, que l'on trie par "cumulative time" (`'cumtime'`).

En exécutant le projet pendant quelques secondes, on peut obtenir des résultats de la forme suivante :

```
         288963 function calls (287734 primitive calls) in 0.605 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
  301/214    0.004    0.000    0.353    0.002 gameview.py:453(do_on_update)
      902    0.011    0.000    0.316    0.000 collision.py:249(check_for_collision_with_lists)
      435    0.004    0.000    0.284    0.001 sprite_list.py:885(update)
      299    0.003    0.000    0.250    0.001 physics_engines.py:752(update)
      136    0.000    0.000    0.246    0.002 gameview.py:269(update)
      266    0.005    0.000    0.245    0.001 gameview.py:204(update)
      299    0.009    0.000    0.244    0.001 physics_engines.py:84(_move_sprite)
      265    0.006    0.000    0.185    0.001 collision.py:137(_get_nearby_sprites)
     8261    0.022    0.000    0.174    0.000 collision.py:91(_check_for_collision)
      265    0.117    0.000    0.124    0.000 query.py:132(__exit__)
     1481    0.003    0.000    0.115    0.000 collision.py:65(check_for_collision)
     3058    0.107    0.000    0.114    0.000 geometry.py:16(are_polygons_intersecting)
     1500    0.019    0.000    0.078    0.000 spatial_hash.py:108(get_sprites_near_sprite)
    14338    0.024    0.000    0.075    0.000 base.py:287(get_adjusted_points)
     2312    0.003    0.000    0.049    0.000 base.py:315(left)
    17686    0.034    0.000    0.047    0.000 base.py:301(_adjust_point)
     2312    0.008    0.000    0.046    0.000 base.py:152(left)
     1462    0.008    0.000    0.038    0.000 collision.py:189(check_for_collision_with_list)
      265    0.002    0.000    0.031    0.000 vertex_array.py:728(transform)
      513    0.003    0.000    0.028    0.000 gameview.py:66(update)
      ...
```

La dernière colonne indique de quelle fonction/méthode on parle.
Les autres colonnes donnent les mesures :

* `ncalls` : le nombre total d'appels à cette fonction ;
* `tottime` : le temps total passé *dans cette fonction* (mais pas dans les fonctions appelées par cette fonction) ;
* `percall` : `tottime / ncalls` ;
* `cumtime` : le temps total passé dans cette fonction *y compris les fonctions qu'elle appelle* ;
* `percall` : `cumtime / ncalls`.

Il est normal de voir `do_on_update` tout en haut de la liste, puisque nous trions par cumulative time, et que l'on mesure uniquement cette fonction.
Il est aussi rassurant de voir qu'elle passe le plus clair de son temps dans `check_for_collision_with_lists`, et non pas dans du code de notre projet.
Cela veut dire qu'en l'état, on peut difficilement faire mieux (à moins d'avoir oublié un `use_spatial_hash` bien placé !).

Une alternative est de trier par total time (`'tottime'`).
Cela peut nous dire quelle fonction prend beaucoup de temps *sans compter* les fonctions qu'elle appelle.
Les fonctions avec un grand `tottime` sont des bonnes candidates à optimiser.

### Flame graphs

La vue tabulaire de `cProfile` est utile, mais elle n'est pas la meilleure.
En 2011, Brendan Gregg a inventé un type de graphique particulièrement adapté à la visualisation des résultats d'un profiler : le [flame graph](Brendan Gregg), ou "graphe en flammes".

Au lieu d'imprimer un tableau à l'écran, on peut demander à `cProfile` d'écrire ses mesures dans un fichier `.prof`.
On peut ensuite utiliser une bibliothèque externe pour créer un flame graph à partir de celles-ci.
À la place du `print_stats`, utilisons :

```python
game_view.profiler.dump_stats("profile.prof")
```

et ré-exécutons le programme pendant quelques secondes.
Cette fois, rien n'est affiché à l'écran, mais un fichier `profile.prof` a été créé.
Celui-ci n'est pas humainement compréhensible en tant que tel.
On va se servir de l'outil externe [SnakeViz](https://jiffyclub.github.io/snakeviz/) pour visualiser son contenu sous forme de flame graph.

Ses instructions d'installation disent d'utiliser `pip install`.
Pour rappel, à chaque fois que c'est le cas, nous utilisons `uv add` à la place.
Comme c'est un outil de développement (et non un outil dont dépend le code de votre application), on utilise l'option `--dev` pour l'installer, comme `pytest` et `mypy`.

```bash
$ uv add --dev snakeviz
```

On peut ensuite le lancer avec

```bash
$ uv run -m snakeviz profile.prof
```

Ceci ouvre une page Web interactive avec le flamegraph.

Vous pouvez voir les résultats de mon test [sur cette page](/files/snakeviz-example/).

[Lisez la documentation de SnakeViz](https://jiffyclub.github.io/snakeviz/) pour comprendre comment interpréter cette visualisation.
Nous en ferons aussi le tour lors du prochain cours.

## Techniques d'optimisation

Lorsqu'on a identifié quelles fonctions sont les plus problématiques, on peut s'attaquer à les optimiser.
Il y a un grand nombre de techniques pour cela.
Certaines seront très spécifiques à Python ; d'autres seront plus transférables à d'autres langages.

On pourrait passer un semestre entier à vous enseigner des techniques d'optimisation.
Je ne prétends donc absolument pas être exhaustif sur le sujet.
Je vous propose uniquement quelques pistes générales.
Il vous reviendra d'explorer plus en détail celles qui vous semblent prometteuses en fonction des mesures que vous aurez relevées.

### Améliorer la complexité

On ne le répétera jamais assez.
La manière la plus efficace d'optimiser un algorithme coûteux est d'en améliorer la complexité.
Nous avons déjà exploré ce sujet en long et en large.
Pour rappel, une petite liste de ce à quoi on peut penser :

#### Éviter de calculer plusieurs fois la même chose

Calculer 2 fois la même chose, ça ne change rien ; la recalculer $n$ fois, c'est problématique.
Évitez de faire plusieurs fois les mêmes calculs.

Dans le cadre de votre projet, cela inclut le fait de faire le même calcul à chaque frame, alors que vous pourriez le faire une seule fois au lancement du programme ou au chargement d'un niveau.

#### Utiliser les bonnes structures de données

Si vous avez une `list` sur laquelle vous faites souvent des tests d'appartenance avec `in`, vous devriez utiliser un `set` ou un `dict`.

De manière générale, choisissez vos structures en fonction des opérations dont vous avez besoin.
Prenez celle qui rend ces opérations efficaces.

#### Utiliser des algorithmes connus

Beaucoup de problèmes peuvent se ramener à des problèmes connus.
Les différents [problèmes de plus court chemin](https://fr.wikipedia.org/wiki/Probl%C3%A8me_de_plus_court_chemin) sont des classiques.
Identifiez si votre problème peut s'exprimer sous forme d'un problème connu, puis réutilisez un algorithme efficace pour le résoudre (potentiellement *via* une bibliothèque externe).

### Éviter le code Python

En Python, malheureusement, dès qu'on a des problèmes de performances brutes, la réponse est souvent la même : *se débarrasser du code Python* !
Python est un des rares langages beaucoup utilisés qui sont complètement *interprétés*.
Cela veut dire que ce n'est pas le CPU qui exécute les instructions que vous écrivez.
Le CPU exécute un *intepréteur*, qui lui *émule* une sorte de CPU pour exécuter une à une vos instructions.

Presqu'aucun langage ne fait ça.
Les autres langages rentrent dans deux catégories :

* les langages compilés *à l'avance* en code machine (AOT pour *ahead of time*) : C++, Rust, OCaml, Zig, etc. ;
* les langages compilés *à la volée* en code machine (JIT pour *just in time*) : Java, Scala, JavaScript, C#, Julia, etc.

Dans un cas comme dans l'autre, c'est bel et bien le CPU qui exécute directement le code, *in fine*.
Certains de ces langages sont plus rapides que d'autres en moyenne, mais cela reste dans un facteur raisonnable, nettement plus petit que 10.
Oui, même JavaScript reste de l'ordre de "seulement" 5 fois plus lent que C++.

Au contraire, Python est dans une toute autre ligue, à être 10 à 100 fois plus lent que les langages ci-dessus.

Il est donc important de limiter le code Python dans les endroits critiques.
Cela passe essentiellement par deux moyens.

#### Utiliser les structures de plus haut niveau

Il vaut mieux utiliser les outils "de plus haut niveau" que Python nous donne.
Par exemple, les [list comprehensions](./structures-de-donnees.html#list-comprehensions) sont plus rapides que des boucles qui font `append` élément par élément, par exemple.
C'est peut-être contre-intuitif pour celles et ceux d'entre vous qui ont l'habitude d'autres langages, dans lesquels c'est l'inverse.
Mais en Python, une list comprehension exécute plus de code "nativement" à l'intérieur de l'interpréteur, plutôt que du code Python.

De même, les fonctions builtins telles que [`sorted`](https://docs.python.org/3/library/functions.html#sorted) seront plus rapides que ce que vous pourriez écrire vous-même en Python.

#### Déléguer à des bibliothèques écrites en C

De nombreuses personnes pensent que "Python est rapide !"
C'est essentiellement dû à la quantité de bibliothèques externes *pour Python* mais *écrites en C*.
L'usage adéquat de ces bibliothèques est critique pour obtenir des performances décentes s'il y a beaucoup de calculs à effectuer.

L'exemple le plus connu, parmi ces bibliothèques, est [NumPy](https://numpy.org/).
Dès que vous avez à faire de nombreux calculs, comme des simulations numériques, des statistiques, des méthodes numériques de résolution d'équations, de l'algèbre linéaire, etc., vous allez vous tourner vers cette bibliothèque.

La série de cette semaine est dédiée à la découverte de cette bibliothèque.

### Éviter les allocations

Une des rares optimisations que vous pouvez faire en Python, et qui se transfère bien à d'autres langages, est d'éviter les allocations dans les endroits critiques.

Pour cela, il faut d'abord comprendre comment est organisée la mémoire vive dans un programme.
Quel que soit le langage ou l'environnement, la mémoire est divisée en deux zones distinctes : la *pile* (*stack*) et le *tas* (*heap*).

La *pile* suit les appels de fonctions.
Lorsqu'on entre dans une fonction, on rajoute sur la pile de l'espace pour les paramètres et les variables locales de cette fonction.
Lorsqu'on quitte la fonction, on "dépile" cet espace, ce qui remet la pile dans l'état précédent l'appel à cette fonction.
À moins de faire de très nombreux appels récursifs, l'espace de la pile est essentiellement gratuit.
Avoir plus ou moins de variables locales dans une fonction ne fait aucune différence.

Le *tas* constitue la majeure partie de la mémoire utilisée par une application.
Quand un programme démarre, le tas est vide.
C'est là qu'on *alloue* dynamiquement la mémoire pour les instances des objets.
En fonction des langages, la syntaxe d'allocation dans le tas varie, mais les concepts restent les mêmes :

* Python : création d'instance comme `Foo(...)`, mais aussi les syntaxes dédiées pour les collections comme `[]`.
* C++ : allocation de mémoire *via* des pointeurs : `new Foo(...)` pour les "pointeurs C" ou `std::make_shared`/`std::make_unique` pour les "smart pointers".
* Java, JavaScript : création d'instance avec `new Foo(...)`.

Les allocations sur le tas ne sont pas gratuites, d'un point de vue performances.

Dans des sections de code critiques pour les performances, on peut chercher à réduire le nombre d'allocations.
Malheureusement, et contrairement à toutes les autres optimisations mentionnées ci-dessus, cela a tendance à *détériorer significativement l'intelligibilité du code*.
Le moyen le plus courant de réduire le nombre d'allocations est de *réutiliser* la même instance pour contenir des données différentes à différents moments.
Cela va directement à l'encontre de tout ce qu'on a appris sur l'usage des données immuables !

Il est donc absolument nécessaire de se lancer dans ce type d'optimisation *uniquement* pour le code que les *mesures* (dont le profiling) ont identifié comme étant particulièrement critique d'un point de vue des performances.
