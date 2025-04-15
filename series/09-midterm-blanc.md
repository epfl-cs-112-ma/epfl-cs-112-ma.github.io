---
title: "Série 9 : Midterm blanc"
layout: default
use_mathjax: true
---

# Série 9 : Midterm blanc

[Solutions](https://github.com/epfl-cs-112-ma/solutions-serie-09)

Cette "série" peut être vue comme un *midterm blanc*.
Il est conseillé de tenter de la résoudre en conditions d'examen :

* Durée : 1h45
* Matériel : uniquement non électronique
* Répondre sur papier

Contrairement au *réel* midterm, celui-ci n'a pas été relu ni résolu par une tierce personne à l'avance.
Il est fort possible que sa durée ne corresponde pas exactement à ce que devrait durer un midterm (je le soupçonne d'être trop long), ou qu'il comporte encore des ambiguïtés.

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Instructions

Plus exactement, voici toutes les consignes officielles qui s'appliqueront au midterm du 17 avril :

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

## Sujet

Dans le jeu *Seven Wonders*, les joueurs et joueuses incarnent des civilisations (anglais : *civilization*).
À chaque tour, chaque civilisation peut *construire une carte* (anglais : *build a card*).
Nous allons modéliser les aspects du jeu nécessaires à calculer le coût d'une telle construction.

Chaque carte a des *ressources requises* (anglais : *required resources*).
En général, pour pouvoir construire une carte, la civilisation doit être capable de produire toutes les ressources requises.
Cependant, plusieurs mécaniques vont permettre de "financer" une carte différemment.

Les ressources sont divisées en deux *catégories* (anglais : *category*) : les matériaux (*material*) et les biens manufacturés (*goods*).
Il y a plusieurs types de ressources dans chaque catégorie.
Par exemple, le bois et la pierre sont des matériaux, et le verre est un bien manufacturé.

Les ressources requises par une carte pourraient, par exemple, être 2 pierres et 1 verre.
On peut voir cela comme un multi-set, ou plus simplement comme une liste de ressources (qui peut donc avoir des doublons).

Chaque civilisation produit *une* ressource fixée.
Par exemple, la civilisation "Pyramides de Gizeh" produit de la pierre.
Elle pourra produire plus de ressources avec des cartes qu'elle aura construites.

Lorsque ses propres ressources ne suffisent pas à construire une carte, une civilisation peut en acheter à ses deux voisines (anglais : *neighbor*).
Elle peut acheter chaque ressource produite par chacune de ses voisines au prix de 2 pièces d'or par ressource (dans la limite de ce qu'elles produisent).

Les cartes *déjà construites* par une civilisation apportent des avantages.
Il y a 2 sortes de cartes :

* Les cartes de *production* (anglais : *production*) produisent une ressource fixée, en un nombre donné d'exemplaires.
  Par exemple, une carte pourrait produire 2 bois (mais pas 1 bois et 1 pierre).
* Les cartes de *commerce* (anglais : *commerce*) réduisent le tarif d'achat des ressources aux civilisations voisines.
  Elles concernent une *catégorie* donnée de ressources, et peuvent soit s'appliquer uniquement à la voisine de gauche, ou uniquement à la voisine de droite, ou aux deux.
  Le coût pour l'achat des ressources dans cette catégorie, à la ou les voisines concernées, est alors seulement de 1 pièce d'or.

Par exemple, une carte de commerce pourrait permettre d'acheter des matériaux à la voisine de droite, pour seulement 1 pièce d'or chaque.

Tous ces éléments vont donc se combiner pour décider si on peut construire une carte.
Et si on le peut, quel est le nombre minimal de pièces d'or à dépenser (on ne s'inquiète pas de savoir combien vont chez chaque voisine ; seulement du total à payer de notre point de vue).

Dernière astuce : certaines cartes peuvent être construites par *chaînage* (anglais : *chain*).
Une carte $A$ peut optionnellement mentionner le *nom* (anglais : *name*) d'une autre carte "de chaînage" $B$.
Si une civilisation a déjà construit une carte $B$, alors elle peut construire $A$ *gratuitement*, sans devoir produire aucune de ses ressources requises.

## Questions

Quand vous définissez des classes, laissez de la place pour y ajouter des méthodes plus tard.
Vous pouvez toujours les ajouter à un autre endroit si vous n'avez plus de place.

En cas de besoin : les fonctions `min` et `max` de Python peuvent être utilisées avec une collection `c` et une fonction de "clef" `key`.
Elle permettent d'obtenir l'élément $x \in c$ tel que $\texttt{key}(x)$ soit minimal/maximal.
Par exemple, `max([5, 3, -6, -4], key=abs)` renvoie `-6`, car `abs(-6)` est maximal.

---

**1.** Définissez un ou des types de données pour représenter les *ressources* et leurs *catégories* respectives.

---

**2.** Définissez un ou des types de données pour représenter les *cartes*.

---

**3.** Définissez un ou des types de données pour représenter les *joueurs*.
N'oubliez pas de représenter les cartes déjà construites.

---

**4.** Écrivez une fonction qui accepte un joueur et une carte, et qui teste si ce joueur peut construire cette carte *par chaînage*.
Elle doit renvoyer `True` si c'est le cas, et `False` sinon.

---

**7.** Écrivez une fonction qui accepte un joueur, et qui retourne sa production totale.

---

**5.** Il sera utile de modéliser un "pool de production" : une réserve de ressources provenant d'une civilisation, et qui n'ont pas encore été consommées.
Il y aura deux sortes de pool : un pool "pour soi" (anglais : *own production*), dont le coût d'utilisation est toujours 0, et un pool "de voisine" (anglais : *neighbor production*), dont le coût va dépendre de quelle voisine il s'agit, et des cartes déjà construites.
Dans tous les cas, un pool contient un nombre variable de ressources, qui commence avec la production totale d'un joueur.

Définissez un ou des types de données pour représenter les pools.

Puis, écrivez une fonction qui accepte une ressource et une liste de pools de production, et qui consomme cette ressource depuis le pool qui l'offre au meilleur prix.
Elle doit renvoyer `None` si aucun des pools ne possède plus cette ressource.
Sinon, elle consomme la ressource depuis le pool le meilleur offrant, puis renvoie le prix auquel la ressource a dû être payée.
Par exemple, si un pool offre cette ressource à 2 pièces d'or, un deuxième à 1 pièce d'or, et le troisième n'en a plus, alors il faut consommer la ressource depuis le deuxième pool et renvoyer 1.

---

**8.** Écrivez une fonction qui accepte un joueur, une carte, et les voisines gauche et droite de ce joueur.
Elle détermine s'il est possible de construire cette carte en combinant les 3 productions.
Si oui, elle renvoie le prix minimal en pièces d'or qu'il faut dépenser (cumulant les dépenses à gauche et à droite).
Si non, elle renvoie `None`.
Cette fonction ne tient pas compte du chaînage.

---

**9.** Finalement, écrivez une fonction qui accepte un joueur, une carte, et les voisines gauche et droite de ce joueur.
Elle se comporte presque comme à la question précédente, à l'exception qu'elle tient compte du chaînage.
Si la carte est construite par chaînage, elle doit renvoyer 0, puisqu'il ne faut rien payer pour exploiter le chaînage.
