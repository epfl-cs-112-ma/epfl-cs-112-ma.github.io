---
title: "Série 5 : Structures de données"
layout: default
use_mathjax: true
---

# Série 5 : Structures de données

[Solutions](https://github.com/epfl-cs-112-ma/solutions-serie-05)

Cette série a pour objectif de pratiquer l'usage des différentes structures de données disponibles en Python, ainsi que les list comprehenions.

Avant de commencer cette série, il est prévu que vous ayez suivi le tutoriel de cette semaine :

* [Structures de données](/tutoriels/structures-de-donnees.html)

Nous vous recommandons de créer [un nouveau projet Python](/references/quick-projet-setup.html) pour chaque série d'exercices.
Cela vous permettra d'isoler plus facilement le contenu des différentes semaines.

Cette série contient 2 exercices :

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Jeu de société

Dans le jeu de société [Brass: Birmingham](https://boardgamegeek.com/boardgame/224517/brass-birmingham), on construit des *industries* sur des *villes*, ainsi que des *chemins de fer* pour connecter les villes entre elles.
Quand on construit une industrie, elle n'est pas encore "rentabilisée".
C'est seulement si on la vend plus tard qu'elle rapportera des points.

En fin d'ère, on gagne des points en fonction :

* des industries que l'on a construites et *rentabilisées* ;
* des industries rentabilisées sur les villes connectées par nos chemins de fer.

Plus précisément :

* Sur le plateau, il y a un certain nombre de *villes* et de *connexions possibles* ;
* Une connexion possible touche 2 villes ;
* Chaque ville a de la place pour construire plusieurs industries ;
* Chaque industrie construite appartient à un joueur ou joueuse ;
* Plusieurs joueurs peuvent construire sur la même ville, et peuvent construire chacun plusieurs industries sur la même ville ;
* Une connexion possible peut être construite par maximum un joueur ou joueuse ;
* Une industrie construite peut être rentabilisée ou non ;
* Chaque industrie a un nombre de points de victoires associé.

On compte les points comme suit.
Chaque joueur ou joueuse gagne :

* la somme des points de victoires associés à toutes les industries qu'elle a construites *et rentabilisées* ; plus
* la somme des points de victoires octroyés par chacune des connexions qu'elle a construites :
    * une connexion rapporte 1 point de victoire par industrie *rentabilisée* dans les deux villes touchées par la connexion (peu importe qui a construit ces industries !).

**Modélisez** la situation de jeu ci-dessus.
Puis **implémentez** la fonction `compute_score(player: Player) -> int` qui calcule le nombre de points gagnés par un joueur ou joueuse.
Essayez de réaliser cette implémentation *sans boucle* (ni fonction récursive), donc en exploitant au maximum les comprehensions.

La fonction [`sum()`](https://docs.python.org/3/library/functions.html#sum) de Python calcule la somme des éléments d'une liste d'entiers.

## Compteur de mots

Pour des analyses lexicographiques, on veut produire un graphe du nombre d'occurrences des mots dans un texte.
Il y a deux fichiers, dont on peut supposer qu'on reçoit le contenu sous forme d'une seule `str` chacun :

* Un fichier contenant le texte à analyser ;
* Un fichier contenant une liste de mots à ignorer (par exemple, des connecteurs), un par ligne.

Vous devez découper le texte à analyser en mots :

* les espaces et les fins de lignes `\n` délimitent les mots ;
* on retire les `.,;:!?` des mots ainsi découpés.

Ensuite, vous devez compter combien de fois apparaît chaque mot dans le texte.
Vous devez éviter de compter les mots à ignorer.
On suppose que le texte a été écrit entièrement en minuscules.

Présentez les résultats en ordre décroissants sous la forme suivante :

```
1234 je
 932 tu
 456 mange
  23 pomme
   1 alambique
```

à savoir :

* un mot par ligne ;
* pour chaque mot, le nombre d'occurrences (aligné à droite), suivi d'un espace, suivi du mot ;
* trié par ordre décroissant.

N'hésitez pas à consulter la documentation de Python, par exemple [la fonction `sorted`](https://docs.python.org/3/library/functions.html#sorted) et [la classe `str`](https://docs.python.org/3/library/stdtypes.html#string-methods).
Si vous n'êtes pas convaincu-e par l'aspect "tout en minuscule", je vous invite à consulter [la fonction `str.casefold`](https://docs.python.org/3/library/stdtypes.html#str.casefold) en particulier.

Faites une analyse de complexité de votre implémentation.

Testez votre programme sur [le texte complet de l'Odyssée d'Homer](https://onlinebooks.library.upenn.edu/webbin/gutbook/lookup?num=14286).
Téléchargez la version "Unicode (664K)".
