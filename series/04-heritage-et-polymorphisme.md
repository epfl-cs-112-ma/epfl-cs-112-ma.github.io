---
title: "Série 4 : Héritage et polymorphisme"
layout: default
use_mathjax: true
---

# Série 4 : Héritage et polymorphisme

[Solutions](https://github.com/epfl-cs-112-ma/solutions-serie-04)

Cette série a pour objectif de pratiquer l'usage de l'héritage, du sous-typage et du polymorphisme.

Avant de commencer cette série, il est prévu que vous ayez suivi le tutoriel de cette semaine :

* [Héritage et polymorphisme](/tutoriels/heritage-et-polymorphisme.html)

Nous vous recommandons de créer [un nouveau projet Python](/references/quick-projet-setup.html) pour chaque série d'exercices.
Cela vous permettra d'isoler plus facilement le contenu des différentes semaines.

Cette série contient 2 exercices :

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Boîte aux lettres

Il s'agit dans cet exercice de proposer une conception modélisant une boîtes aux lettres.

Une boîte aux lettres recueille des lettres, des colis et des publicités.

Une lettre est caractérisée par :

* son poids (en grammes) ;
* le mode d'expédition (express ou normal) ;
* son adresse de destination ;
* son format ("A3", "A4" ou "A5").

Un colis est caractérisé par :

* son poids (en grammes) ;
* le mode d'expédition (express ou normal) ;
* son adresse de destination ;
* son volume (en litres).

Une publicité est caractérisée par :

* son poids (en grammes) ;
* le mode d'expédition (express ou normal) ;
* son adresse de destination.

Voici les règles utilisées pour affranchir le courrier :

* En mode d'expédition normal :
  * Le montant nécessaire pour affranchir une lettre dépend de son format et de son poids : \
    Formule : $\text{montant} = \text{tarif de base} + 1.0 \cdot \text{poids (kilos)}$. \
    Le tarif de base pour une lettre "A5" est de 1.50, de 2.50 pour une lettre "A4", et 3.50 pour une lettre "A3".
  * Le montant nécessaire pour affranchir une publicité dépend de son poids : \
    Formule : $\text{montant} = 5.0 \cdot \text{poids (kilos)}$.
  * Le montant nécessaire pour affranchir un colis dépend de son poids et de son volume : \
    Formule : $\text{montant} = 0.25 \cdot \text{volume (litres)} + 1.0 \cdot \text{poids (kilos)}$.
* En mode d'expédition express : les montants précédents sont doublés, quelque soit le type de courrier.
* Seul le courrier valide est affranchi.
* Un courrier n'est pas valide si l'adresse de destination est vide.
* Un colis n'est pas valide si son adresse de destination est vide ou s'il dépasse un volume de 50 litres.

Les trois méthodes principales liées à la boîte aux lettre sont les suivantes :

1. Une méthode `frank()` permettant d'associer à chaque courrier de la boîte, le montant nécessaire pour l'affranchir (*frank* en anglais).
   Cette méthode retournera le montant total d'affranchissement du courrier de la boîte.
2. Une méthode `invalid_mails()` calculant et retournant le nombre de courriers invalides présents dans la boîte aux lettres.
3. Une méthode `display()` qui retourne une `list[str]` représentant le contenu de la boîte aux lettres tel qu'on l'afficherait à l'écran.
   Chaque élément de la liste représente une ligne, correspondant à un courrier.
   On veillera à inclure les informations utiles, y compris si le courrier est valide.

Sur papier, commencez par lister les classes dont vous aurez besoin, ainsi que des relations d'héritage/sous-typage éventuelles, afin de mettre en œuvre la conception suggérée en tenant compte des contraintes mentionnées.
Ensuite, **implémentez et testez** vos classes de manière adéquate.

Vos classes doivent éviter de dupliquer inutilement des méthodes ou des attributs.
Lesquelles de vos classes devraient être muables ou immuables ?

Veillez à respecter le LSP.

**Réflexion bonus :** quel type de données est approprié pour représenter les prix ?

## Cadastre

Avec toutes les nouvelles constructions en cours, la ville d'Écublens VD peine à gérer son cadastre.
Par cadastre, on entend ici l'inventaire des bâtiments et de leurs propriétaires (c'est une simplification).
Nous allons l'aider.

Il y a deux types de bâtiments (*building*) : les maisons (*house*) et les immeubles (*appartment building*).

Les maisons sont caractérisées par une surface habitable (en m²), un nombre de pièces, et une taille de jardin (en m²).

Les immeubles sont constitués de plusieurs appartments (*appartment*).
Chaque appartement est caractérisé par une surface habitable (en m²), un nombre de pièces, et une taille de balcon (en m²).

Chaque bâtiment a un ou une propriétaire (*owner*), qui est une personne.
Une personne est caractérisée par son nom complet.
Une personne peut posséder plusieurs bâtiments.

Un bâtiment peut être vendu et donc changer de propriétaire.
En revanche, on supposera qu'aucuns travaux ne seront nécessaires une fois les bâtiments construits.

La ville veut pouvoir calculer à combien taxer les propriétaires.
Chaque bâtiment est taxé selon la formule suivante :

$$ \text{Impôt} = \text{TauxA} \cdot \text{surface habitable} + \text{TauxB} \cdot \text{surface de jardin} $$

Les valeurs cette années sont de $\text{TauxA} = 5.6$ et $\text{TauxB} = 1.5$.
La surface habitable d'un immeuble est la somme des surfaces habitables de tous ses appartements.

**Concevez et implémentez** un système de classes permettant de représenter les concepts ci-dessus.
La ville voudra pouvoir produire un raport de l'impôt à imputer à chaque propriétaire.

Sur papier, commencez par lister les classes dont vous aurez besoin, ainsi que des relations d'héritage/sous-typage éventuelles, afin de mettre en œuvre la conception suggérée en tenant compte des contraintes mentionnées.

Vos classes doivent éviter de dupliquer inutilement des méthodes ou des attributs.
Lesquelles de vos classes et/ou de leurs attributs devraient être muables ou immuables ?

Veillez à respecter le LSP.
