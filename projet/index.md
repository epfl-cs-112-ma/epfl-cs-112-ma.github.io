---
title: "Projet : Platformer 2D"
layout: default
---

# Projet : Platformer 2D

![Screenshot bande annonce](/assets/img/projet/trailer-screenshot.png)

Le but du projet de cette année est de réaliser un peu jeu de type "platformer" en 2D.
[Super Mario Bros](https://supermarioplay.com/fr), publié par Nintendo en 1985, est l'exemple canonique de ce type de jeu.
À l'époque, il s'agissait d'une prouesse technologique.
En 2025, vous allez pouvoir produire en quelques mois un jeu d'ampleur similaire.

Nous n'allons pas partir de rien !
Nous allons profiter de la bibliothèque [Arcade](https://api.arcade.academy/en/latest/) pour nous donner quelques primitives graphiques et physiques.
Vous pourrez également utiliser des images et sons (*assets*) tout faits, libres de droits.
Cependant, vous allez développer toute la logique du jeu.

Vous recevrez chaque semaine des instructions pour aller un peu plus loin dans le développement du jeu.
Au fur et à mesure des semaines, vous disposerez de plus en plus de libertés sur les choix de conception, d'implémentation, et même des fonctionnalités que vous ajouterez à votre de jeu.
La première semaine sera détaillée pas à pas, avec peu de libertés, afin que vous maîtrisiez les principaux concepts impliqués dans la réalisation d'un jeu de ce type.
Lors des deux dernières semaines, vous aurez pratiquement quartier libre pour faire briller votre créativité.

## Étapes de la réalisation du projet

* Semaine 1 – Découverte d'Arcade : personnage, murs fixes, pièces à collecter et caméra (à venir jeudi matin)

## Administration

### Rendu

Le projet est à réaliser impérativement en groupe de deux personnes sans aucune exception.
Les groupes sont à constituer d'ici au 5 mars 2025.

Concernant le rendu, nous ne vous demandons pas de rapport (document) sur le projet.
Par contre, nous vous demandons de commenter abondamment et intelligemment les codes sources que vous nous rendrez (les commentaires remplaceront le rapport) et de fournir quatre fichiers complémentaires :

* Un fichier `LOG.md` (journal) résumant votre progression et reportant, brièvement, semaine après semaine, ce que vous avez fait sur le projet.
  Voir un [exemple ici](./LOG.md).
  Cela permettra à votre groupe de mieux se coordonner et de toujours savoir où vous en êtes.
  Soyez concis mais précis, clairs et objectifs.
  On peut très bien comprendre que, ponctuellement, telle ou telle semaine vous ayez eu d'autres contraintes et ayez moins avancé sur le projet.
  Vous pouvez également inclure vos états-d'âme, tout en respectant celles et ceux qui vous liront.

  Veillez donc à bien mettre à jour votre JOURNAL chaque semaine (cela fera partie de la notation).

* Un fichier `ANSWERS.md` répondant point par point aux questions posées dans les instructions hebdomadaires du projet.
  Ces réponses ne sont bien sûr pas uniques mais dépendent de la façon dont *vous* concevez votre projet.
  Elles serviront aux correcteurs et correctrices à mieux comprendre votre conception.

* Un fichier `DESIGN.md` ou `DESIGN.pdf` (à faire au fur et à mesure) décrivant vos modules, vos classes, leurs responsabilités et leurs interactions.
  On s'attend évidemment ici à ce que vous mettiez en valeur la façon dont vous avez mis en œuvre les différentes méthologies vues au cours (elles ne seront pas forcément toutes applicables).

  Ce fichier peut être soit sous forme textuelle donnant simplement la liste des classes et leur relations, soit sous forme graphique (en utilisant par exemple [Inkscape](https://inkscape.org/fr/)).

* Un fichier `README.md` expliquant ce qu'est votre programme et comment l'utiliser.
  Donnez quelques exemples concrets.
  Imaginez que c'est le fichier que votre famille et vos amis vont consulter pour découvrir votre jeu (bon, des amis qui ont installé `uv`...).

La date limite de remise du projet est le vendredi 30 mai à 23h58.
Cette date est absolument impérative.
Aucune extension de délai ne sera accordée !

### Organisation du travail

Ce projet constitue une part importante de l'évaluation du cours (la moitié) et est spécifié de façon assez large pour permettre un travail créatif.
Le "danger" est donc grand de se laisser submerger par ce projet.
C'est d'autant plus vrai que, d'expérience, le développement d'un jeu est très motivant.
Il faut évidemment éviter cela à tout prix !
Voici donc plusieurs conseils et remarques afin d'organiser au mieux votre travail, dans une harmonie maximale avec toutes les autres contraintes de votre semestre.

Citant ici la [présentation générale du cours](/presentation.html) :

> * Pour bien apprendre la programmation, il faut *pratiquer*.
>   Cela ne se fait pas du jour au lendemain et demande en effet une certaine quantité de travail *et* une certaine régularité.
> * Néanmoins cette quantité de travail doit rester "raisonnable" : entre 5 et 8 heures *grand maximum* par semaine *tout compris* (tutoriels + cours + exercices + travail personnel + projet).
> * Il s'agit là d'une branche de semestre pour laquelle vous ne travaillez plus du tout après la fin du semestre.
>   La période sur laquelle vous devez fournir le travail pour cette branche étant plus courte, il est évident qu'à charge totale égale, la charge par semaine devient plus grande.
> * Concernant le projet, celui-ci est également là pour vous apprendre
>     * à répartir le travail dans votre binôme (ne faites donc pas le travail en double) ;
>     * à faire ce que vous pouvez en un temps imparti (au lieu de faire le maximum en un temps infini).
> * Fixez-vous un objectif atteignable pour votre niveau : quel est votre objectif ? 6.0 ?
>   Certains en font beaucoup trop (en particulier sur le projet), principalement en raison de la stimulation des autres groupes (dont certains passionnés).
>   La moyenne des projets rendus est souvent très élevée (5 à 5.5).
>   Pour quelqu'un qui a un niveau moyen de 3.5 à 4.5, fournir le travail de développer un projet à un niveau de 5.5 à 6.0 est simplement colossal !
>   Ce n'est pas ce que je vous demande...
>   Faire un exercice ne signifie pas nécessairement le réussir à 100 % et obtenir tous les points.
>   Ceci dit, pour obtenir finalement un 4.5, il vaut mieux viser au-dessus (p.ex. 5.0).

Si donc certains passent effectivement trop de temps sur ce projet, et, pire !, au détriment d'autres branches, c'est essentiellement pour les raisons suivantes.
Cela dit, je tiens à signaler ici qu'il y a chaque année un grand nombre d'étudiantes et étudiants qui arrivent très bien à gérer ce projet et considèrent que c'est une charge lourde certes, mais justifiée, non excessive et enrichissante.

1. une mauvaise répartition du travail entre binômes et dans le temps ;
2. une mauvaise évaluation du travail à effectivement fournir dans ce projet ;
3. une mauvaise gestion des contraintes et priorités.

Pour 1. : Le projet est un travail que je vous demande d'effectuer en binôme (ne faites donc pas le travail en double) et de commencer dès le début du semestre.

Pour 2. : Le projet est en effet conçu dès le départ pour laisser une large place à la création et au développement personnel.
Il est de plus prévu pour une population extrêmement hétérogène.
Plusieurs ont donc tendance à trop en faire : trop par rapport aux attentes minimales du cours et trop par rapport à leur propre niveau en programmation.
Mes collègues ont constaté chaque année chez certaines et certains une implication dans le projet disproportionnée par rapport à leur niveau effectif.
Faites ce que vous pouvez en un temps imparti, au lieu d'essayer de faire le maximum en un temps infini !
Fixez-vous un objectif raisonnable, atteignable pour votre niveau.

Le point 3. (gestion des contraintes et choix des priorités), est une conséquence du point précédent : ne sachant se limiter raisonnablement dans le projet, certains s'investissent trop dans cette branche de semestre au détriment d'autres branches plus fondamentales pour leur Section.
Cela relève plus largement de l'organisation globale de leurs études et de la bonne gestion des contraintes que la charge de travail globale, à laquelle ils n'étaient pas habitués au Gymnase, leur impose.

Donc, pour résumé à ce stade, je dirais : faites attention à bien vous organiser et à ne pas vous laisser noyer par le projet.
C'est dans ce sens que je vous donne les conseils d'organisation suivants.

Ce projet est prévu pour être réalisé partiellement pendant les séries d'exercices et partiellement sous forme de travail personnel indépendant.
Comme la programmation complète du projet n'est évidement pas envisageable sur un laps de temps réduit aux séances d'exercices, vous ne pourrez réaliser lors de ces séances que des portions de l'ensemble du projet, le reste devant être fait "à la maison".
Profitez par contre des séances d'exercices pour poser des questions aux assistantes et assistants.

Par ailleurs, je vous conseille de vous assurer avoir bien compris les concepts présentés dans les tutoriels et en cours avant de vous lancer dans la réalisation de la nouvelle partie correspondante du projet.
Il est conseillé de faire au préalable quelques exercices proposés dans les séries.
Une fois les concepts bien compris, la réalisation du projet constitue bien sûr aussi un entraînement et une révision pour les examens.

Ce projet est prévu pour être réalisé par binômes.
Il est important de bien répartir entre vous les tâches à accomplir chaque semaine.
Il est évident que vous ne devez pas chacune/chacun développer l'entièreté des tâches du projet.
Essayez donc de rapidement vous organiser, concevoir correctement la répartition des tâches et utiliser une conception modulaire (modules, interfaces, encapsulation, et tests) afin de diminuer votre charge de travail individuelle.

Un des buts du projet est justement de vous apprendre à développer du code en parallèle et de façon complémentaire.
Par contre, jetez quand même un coup d'œil à la partie développée par votre partenaire et discutez-en ensemble.
Vous apprendrez sûrement beaucoup de ce genre de pratique.

Enfin, pour vous faciliter la vie durant le projet :

* Travaillez avec régularité et ne prenez pas trop de retard. C'est une illusion que de croire pouvoir le rattraper par la suite.

* Pensez à créer des commits régulièrement, afin de sauvegarder les états "stables" de votre projet.

* Testez !
  Écrivez des tests pytest pour tout ce qui est un peu délicat.

* Commencez rapidement !

* Réfléchissez avant de commencer à programmer.
  Prenez le temps avec votre collègue de bien réfléchir à comment coder, quelles structures de données utiliser, etc.
  Cela facilitera grandement la programmation par la suite et vous évitera d'avoir à refaire plusieurs fois la même chose.

* Modularisez votre code.

* Commentez votre code au fur et à mesure : il est quelque peu ingrat (pour ne pas dire plus) de devoir commenter un grand nombre de lignes de codes la veille de la remise.
  On ne se rappelle alors plus ce que l'on avait voulu faire.
  Les commentaires aident à la correction et, en ce sens, participent à la notation.

* Minimisez le nombre d'erreurs mypy actives en même temps.
  Profitez du fait que VS Code vous montre toutes les erreurs de syntaxe et de types en live pour les corriger au fur et à mesure.
  Il est plus facile de corriger 2 ou 3 fautes de syntaxe dans quelques nouvelles lignes de code que 100 fautes dans 400 lignes de code !

* Utilisez dans votre code des identificateurs porteurs de sens.
  Ne faites pas d'abbréviations.
  VS Code vous propose de toutes façons de compléter les identificateurs lors de l'écriture.
  Cela permet de s'y retrouver plus facilement par la suite (reprise de code, correction d'erreurs et... évaluation par les correcteurs !).
  Le code est *lu* environ 10 fois plus souvent qu'il n'est *écrit*.
  Optimisez donc pour la lecture.

* N'hésitez pas à poser des questions durant les séances d'exercices et sur le forum (de façon privilégiée).
  Par contre, **ne postez aucun code du projet sur le forum** (des pénalités seront appliquées aux groupes qui le font).

* N'hésitez pas non plus à discuter de vos éventuelles difficultés avec les assistantes, assistants et moi-même.

* Faites ce que vous pouvez en un temps imparti (que vous vous serez fixé au départ), au lieu d'essayer de faire le maximum en un temps infini !
  Fixez vous un objectif raisonnable, atteignable pour votre niveau.

* Certains en font beaucoup trop sur le projet, principalement en raison de la stimulation des autres groupes (dont certains passionnés).
  La moyenne des projets finalement rendus est souvent très élevée (5 à 5.5).
  Il est donc clair que pour quelqu'un qui a un niveau moyen de 3.5 à 4.5, fournir le travail de développer un projet à un niveau de 5.5 à 6.0 est simplement colossal !
  Ce n'est pas ce qu'on vous demande...

### Fraude

L'échange *d'idées* entre groupes ou avec des tiers est autorisée (et même recommandée).
Par contre, l'échange *de code* est strictement interdit (cela inclut la diffusion de code sur le forum) !
Le plagiat de code, de quelque façon que de soit et quelle qu'en soit la source, sera considéré comme de la fraude (en plus, c'est illégal et passible de poursuites pénales).

En cas de fraude, vous recevrez la note « NA », soit pour le projet, soit même pour l'entièreté du cours.
Vous serez de plus dénoncés et punis suivant l'ordonnance sur la discipline.
Je me permets également de vous rappeler le code d'honneur et le code de déontologie dont vous avez pris connaissance au début de vos études.

Dans tous les cas, il est fortement conseillé de pouvoir expliquer toutes les lignes de code de son projet.

### Intelligences artificielles

L'usage d'intelligence artificielle *n'est pas interdit*.
En revanche, je vous déconseille très fortement d'utiliser de l'IA pour *produire* (que ce soit du code ou des designs).
Servez-vous de l'IA pour vous documenter, pour comprendre un message d'erreur, etc.
Mais *pas* pour produire du code.

> La calculatrice est faite pour celles et ceux qui savent déjà calculer.
> Il en va de même de l'intelligence artificielle : elle est faite pour celles et ceux qui savent déjà programmer.

Rappelez-vous que le projet est aussi là pour vos faire pratiquer, à grande échelle, les concepts du cours.
Ces concepts seront évalués de manière individuelle lors des examens.
