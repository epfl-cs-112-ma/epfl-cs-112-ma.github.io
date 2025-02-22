---
title: Classes et méthodes
layout: default
use_mathjax: true
---

# Classes et méthodes

La semaine dernière, nous avons déjà rencontré des classes, sans vraiment rentrer dans les détails.
Nous nous sommes contentés d'utiliser des "dataclasses", qui correspondent moralement aux `struct`s de C++ :

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: int
    y: int
```

Il est désormais temps de découvrir le concept plus large de *classe*.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Classes

### Une classe vide

Une *classe* définit les caractériques communes d'un ensemble *d'objets*.
Dans sa version la plus simple, on peut définir une classe sans aucune caractéristique :

```python
class EmptyClass:
    pass
```

Pour créer une *instance* d'une classe, on utilise la même syntaxe que pour l'appel de fonction : `EmptyClass()`.

```python
emptyObject = EmptyClass()
```

En fait, une classe est bien une fonction, qui retourne une nouvelle instance d'elle-même.

Chaque classe définit implicitement un *type* associé.
Le type `EmptyClass` représente l'ensemble de toutes les références vers des instances d'`EmptyClass`.
La variable `emptyObject` a donc pour type `EmptyClass`.
Elle contient en effet une référence vers une instance de cette classe.
On pourrait écrire explicitement :

```python
emptyObject: EmptyClass = EmptyClass()
```

### Attributs et constructeur

Une classe vide ne sert pas à grand chose.
Toutes les instances d'une telle classe se ressemblent, et on ne peut pas faire grand chose avec.
La plupart des classes définissent donc des *attributs donnée*, ou *champs*, pour leurs instances.

Un attribut doit être *déclaré* au niveau de la classe, et *initialisé* au sein d'une fonction un peu particulière : le *constructeur*, qui porte le nom magique `__init__`.

```python
class Person:
    name: str # déclaration de l'attribut avec son type

    # constructeur
    def __init__(self) -> None:
        self.name = "Arthur"
```

La fonction `def __init__` est le *constructeur* de la classe `Person`.
Remarquez qu'elle est définie *à l'intérieur* du corps de la classe (délimité par l'indentation, comme d'habitude).
Le constructeur est appelé pour *initialiser* toutes les nouvelles instances de `Person`.
Lors d'une construction telle que

```python
Person()
```

les étapes suivantes se produisent :

1. Une nouvelle instance de `Person` (une nouvelle boîte dans la mémoire) est créée, et Python retient une référence interne vers cette instance, que nous appellerons $r$.
2. La fonction `__init__` définie à l'intérieur de `Person` est appelée, avec $r$ pour valeur du paramètre `self`.
3. Le corps de cette fonction se déroule comme vous en avez l'habitude.
   L'assignation `self.name = "Arthur"` initialise l'attribut `name` de l'instance à la chaîne `"Arthur"`.
4. Une fois le constructeur terminé, on renvoie $r$ comme résultat de l'appel de `Person()`.

On reçoit donc une référence vers l'instance qui vient d'être créée, et qui a été correctement initialisée.
Si on stocke cette référence dans une variable `p`, on peut ensuite afficher le contenu de son attribut `name`.

```python
p = Person()
print(p.name) # affiche 'Arthur'
```

> Remarque : le nom `__init__` est magique.
> Le constructeur doit avoir exactement ce nom.
> Le nom du paramètre `self`, en revanche, n'est pas magique.
> C'est cependant une convention partagée par toutes les développeuses et développeurs Python du monde.
> N'y dérogez pas !

Les constructeurs peuvent avoir d'autres paramètres que `self`.
Les valeurs pour ces paramètres doivent alors être donnés lors de la construction, à l'appel de la classe :

```python
class Person:
    name: str # déclaration de l'attribut avec son type

    # constructeur
    def __init__(self, name: str) -> None:
        self.name = name

p = Person("Arthur")
```

Notez que nous n'avons pas donné de type explicite à `self`.
Étant donné le rôle du constructeur, Python (et surtout mymy) sait qu'il doit être de type `Person`.
On aurait pu écrire `self: Person`, mais par convention, on ne le fait pas.

### Méthodes

L'intérêt des classes, par rapport aux `struct`s, est qu'elles ne sont pas faites que d'attributs.
Les classes portent aussi des *méthodes*.

Pour les besoins pédagogiques, bien que ce soit [totalement faux](https://shinesolutions.com/2018/01/08/falsehoods-programmers-believe-about-names-with-examples/), supposons que chaque personne possède un prénom et un nom de famille, et que son "nom complet" se compose de la concaténation des deux.
Si on doit stocker le prénom et le nom de famille, il semble superflu de stocker également le nom complet.
On veut donc dériver le nom complet au moyen d'une fonction.

Avec les outils que vous connaissez jusqu'ici, nous serions tentés de définir une fonction indépendante, comme ceci :

```python
class Person:
    first_name: str
    last_name: str

    def __init__(self, first_name: str, last_name: str) -> None:
        self.first_name = first_name
        self.last_name = last_name

def person_full_name(person: Person) -> str:
    return f"{person.first_name} {person.last_name}"

p = Person("Arthur", "Weasley")
print(person_full_name(p)) # affiche "Arthur Weasley"
```

Cette façon de faire *fonctionne*, bien sûr, mais a plusieurs désavantages.
Le désavantage le plus frappant à ce stade est que le nom de ces fonctions devient vite verbeux afin d'éviter les collisions.
Le préfixe `person_` n'est pas utile pour l'humain qui lit ce code.
Il n'est utile que pour distinguer cette fonction d'autres fonctions `full_name` qui agiraient sur d'autres types de données.

On introduit donc le concept de *méthode* : une fonction qui est liée aux instances d'une classe.
Une méthode est définie à l'intérieur d'une classe, et prend un premier paramètre nommé `self` (par convention).
L'appel d'une telle fonction se fait avec la notation `instance.method(arguments)` :

```python
class Person:
    ... # comme avant

    def full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"

p = Person("Arthur", "Weasley")
print(p.full_name()) # affiche "Arthur Weasley"
```

Lors d'un appel à une méthode comme `p.full_name()`, les étapes suivantes se produisent :

1. On identifie la classe de l'objet référencé par `p` ; ici c'est la classe `Person`.
2. Dans cette classe, on cherche une méthode (fonction) appelée `full_name`.
3. On appelle cette fonction avec `p` pour valeur du paramètre `self`.
4. La fonction s'exécute comme d'habitude.

À nouveau, on a omis le type du paramètre `self`, qui est implicitement `Person`.

Bien sûr, tout comme pour le constructeur, une méthode peut avoir des paramètres supplémentaires.
Ceux-ci sont donnés entre parenthèses lors de l'appel de la méthode.
Puisque `self` est une référence vers une instance de `Person`, on peut l'utiliser pour appeler d'autres méthodes de la même classe.

```python
class Person:
    ... # comme avant

    def greet(self, greeting: str) -> str:
        return f"{greeting} {self.full_name()} !"

p = Person("Arthur", "Weasley")
print(p.greet("Hello")) # affiche "Hello Arthur Weasley !"
```

### Commentaires

#### Vous avez déjà utilisé des méthodes

Avez-vous remarqué que la syntaxe de l'appel de méthode est exactement la même que `liste.append(x)`, ou même que `vecteur.push_back(x)` en C++ ?
Ce n'est pas un hasard.
Ces fonctions, que nous avions appelées "fonctions spécifiques" au premier semestre, ne sont rien d'autre que des *méthodes*.
Et `list`/`std::vector` sont, en fait, des *classes* !

#### Comparaison avec d'autres langages

Si vous connaissez déjà les concepts de classes et de méthodes dans d'autres langages, notez les particularités suivantes en Python :

* Le paramètre `self` correspond à ce que beaucoup de langages appellent `this`, qui est défini implicitement.
  En revanche, il est toujours explicite dans la définition des fonctions en Python.
* De même, l'accès aux attributs et aux autres méthodes de `self` requiert explicitement le préfixe `self.`.
  On ne peut pas se contenter de `full_name()` dans `greet` ; cela chercherait à appeler une fonction globale appelée `full_name`.

#### Déclaration des attributs

J'ai dit plus haut que les attributs *doivent* être déclarés au niveau de la classe.
J'ai menti.
En Python, c'est l'initialisation dans le constructeur qui fait foi.
Mypy est capable d'inférer le type des attributs d'après le type de valeur qui leur est assignée dans le constructeur.
Cependant, cela rend le code moins lisible.
Je recommende donc très fortement de systématiquement déclarer tous les attributs des classes en amont.

## Encapsulation

Avec l'ajout des méthodes, une nouvelle opportunité se dessine pour mieux concevoir des programmes : *l'encapsulation*.
En termes très générique, l'encapsulation est la capacité de *cacher les détails* d'implémentation, et par là de *protéger ses invariants* des influencent externes.
L'encapsulation est étroitement liée à la notion *d'interface*.

### Exemple de la vie réelle

Commençons par un exemple de la vie réelle : un distributeur de billets de banques.

#### Interface

Un distributeur expose une certaine *interface* aux clients d'une banque.
Il y a un endroit pour insérer sa carte de banque, un pavé numérique pour entrer le code secret, un écran qui permet de sélectionner le montant que l'on désire retirer, et une sortie pour les billets.
Lorsque vous interagissez avec le distributeur, vous avez seulement besoin de savoir comment manipuler cette interface.
Vous savez que si vous suivez le protocole, vous recevrez votre argent (le résultat attendu) :

1. Insérer la carte
2. Indiquer le code PIN
3. Sélectionner le montant sur l'écran
4. Valider
5. Récupérer la carte
6. Récupérer les billets

Vous savez aussi que si quelque chose ne se passe pas bien, vous recevrez un message d'erreur (une exception).
Quelques exemples :

* Mauvais PIN
* Demande de plus d'argent qu'il n'est disponible
* Attendre trop longtemps avant de récupérer la carte

Tout ce protocole constitue *l'interface* du distributeur.
C'est le *contrat* qu'il vous propose.
Si vous respectez votre part du contrat (les *préconditions*), vous avez la garantie de recevoir votre argent (les *résultats*).

Remarquez que vous n'avez absolument pas besoin de savoir *comment* le distributeur remplit sa part du contrat.
Vous ne connaissez pas *l'implémentation* du dstributeur ; pourtant vous êtes parfaitement en mesure de l'utiliser.
D'ailleurs, il y a fort à parier que les distributeurs d'UBS fonctionnent différemment de ceux de la BCV.
En tant que cliente ou client, ça n'a aucun impact sur la façon dont vous les utilisez.

C'est ça qui fait la force d'une bonne interface : un ensemble de *paramètres* (carte, code PIN, montant), un *résultat* attendu (les billets) ou des cas *d'exceptions* (mauvais PIN, etc.), et surtout un *contrat* qui relie toutes ces choses ensemble.

#### Encapsulation

Le distributeur a aussi un rôle *d'encapsulation*.
L'interface vous *permet* d'utiliser le distributeur en ne connaissant que son contrat.
L'encapsulation, quant à elle, vous *empêche* d'utiliser le distributeur en dehors du contrat.

L'implémentation du distributeur a probablement accès à un coffre-fort de billets de banques.
(Ou alors, pour ce qu'on en sait, il les fabrique au vol ; l'interface fait que nous ne savons pas !)
Pourtant, même si vous interagissez avec le distributeur, et que lui interagit avec le coffre-fort, *vous n'avez pas accès* au coffre-fort directement.
Même si vous essayez de ne pas suivre le contrat, le distributeur ne vous laisse pas faire.
Le distributeur *encapsule* le coffre-fort.

Notez que sans interface claire, il ne pourrait pas y avoir d'encapsulation (sans perte de fonctionnalité).
Si je ne connaissais pas un contrat bien définit qui s'affranchit de l'implémentation, je serais obligé d'avoir accès au coffre-fort pour être convaincu que je recevrai mes billets.

Ces deux principes, interface et encapsulation, sont déterminants pour l'écriture de programmes de moyenne à grande taille.
Les classes et leurs méthodes nous donnent l'opportunité de présenter une interface aux utilisateurs de la classe, et d'encapsuler les détails d'implémentation.
C'est l'essence de la programmation orientée objet.

### Méthodes et interface

Commençons par utiliser les méthodes pour donner une vie à la notion d'interface dans nos programmes.
Imaginons une classe `CashDispenser` dont les instances seraient des distributeurs de billets.
Nous voulons modéliser nos interactions au moyen de méthodes qui fournissent un contrat clair.

Commençons par une méthode simple qui nous *indiquerait* seulement combien d'argent nous avons sur notre compte.
Voici une proposition :

```python
class CashDispenser:
    def available_amount(self, card: BankCard, pin: int) -> int:
        pass # not shown
```

Cette définition est-elle suffisante pour savoir comment utiliser cette méthode, sans savoir comment elle fonctionne ?
Malheureusement non !
Tout ce que nous avons de manière sûre, c'est le type des paramètres et de la valeur de retour.
Nous ne savons pas exactement ce que leurs *valeurs* représentent.
Par exemple, quelle est la monnaie de la valeur de retour ?
Se chiffre-t-elle en CHF entiers, ou en centimes de CHF ?
Pire, nous ne savons pas sous quelle conditions l'appel est valide, et ce qu'il se passe si ce n'est pas le cas.

C'est là qu'intervient la *documentation*.
La *docstring* de la méthode sert à spécifier le contrat qui relie le distributeur, et les arguments, et les résultats possibles.
Une version plus acceptable serait donc :

```python
class CashDispenser:
    def available_amount(self, card: BankCard, pin: int) -> int:
        """Demande combien d'argent est disponible sur la carte donnée.

        Le code PIN doit être un entier de 4 à 6 chiffres. Si ce n'est pas le
        bon code PIN, une exception de type `ValueError` est déclenchée.

        Le résultat est exprimé en centimes de CHF.
        """
        pass # not shown
```

La documentation d'une méthode fait donc partie intégrante de *l'interface*.
Sans documentation, il n'y a pas de contrat, et sans contrat, il n'y a pas d'interface.

> Question : Pourquoi a-t-on choisi un `int` exprimé en centimes ici, et pas un `float` exprimé en CHF ?

### Méthodes et encapsulation

Revenons au problème plus complexe de retirer de l'argent.
Voici une interface acceptable pour cette méthode :

```python
class CashDispenser:
    def withdraw(self, card: BankCard, pin: int, amount: int) -> list[BankNote]:
        """Retire un montant donné d'un compte bancaire, sous forme de billets.

        Le code PIN doit être un entier de 4 à 6 chiffres. Si ce n'est pas le
        bon code PIN, une exception de type `ValueError` est déclenchée.

        La quantité demandée est exprimée en centimes de CHF.

        Si le compte de la carte n'a pas suffisamment d'argent, une exception
        de type `NotEnoughMoneyError` est déclenchée. Si le coffre-fort ne
        dispose pas des billets nécessaire pour former la quantité demandée,
        une exception de type `MissingBankNotesError` est déclenchée.

        En cas de succès, le montant disponible sur le compte désigné par la
        carte est réduit de `amount`.
        """
        pass # not shown
```

Illustrons maintenant les aspects d'encapsulation.
Pour cela, imaginons une façon *d'implémenter* cette méthode.
Nous aurons besoin de deux attributs dans la classe `CashDispenser`.

```python
class CashDispenser:
    account_manager: AccountManager
    safe: Safe

    def withdraw(self, card: BankCard, pin: int, amount: int) -> list[BankNote]:
        """..."""
        self.account_manager.validate_pin(card, pin) # may throw ValueError
        self.account_manager.decrease_money(card, amount) # may throw NotEnoughMoneyError
        bank_notes = self.safe.get_notes(amount) # may throw MissingBankNoteError
        return bank_notes
```

Il faudrait évidemment voir les contrats respectifs des trois méthodes que nous appelons (`validate_pin`, `decrease_money` et `get_notes`) pour juger si cette implémentation est correcte ou non.
Nous laisserons ces spécifications en exercice.

> Un problème se cache cependant dans cette implémentation.
> Voyez-vous lequel ?
> Comment le corrigeriez-vous ?

Un code client de cette classe pourrait ressembler à

```python
try:
    dispenser = bank.get_cash_dispenser()
    bank_notes = dispenser.withdraw(my_card, my_pin, amount=50)
    print(bank_notes)
except Error as e:
    print(e)
```

qui serait conforme au contrat.

Mais un code malicieux, ou simplement inattentif, pourrait être tenté d'écrire à la place :

```python
try:
    dispenser = bank.get_cash_dispenser()
    bank_notes = dispenser.safe.get_notes(amount=50)
    print(bank_notes)
except Exception as e:
    print(e)
```

Ceci compromettrait évidemment l'intégrité de la banque, qui se retrouverait bien vite en faillite.

C'est ici qu'intervient *l'encapsulation*.
Bien qu'une instance de `CashDispenser` ait besoin d'utiliser son `safe: Safe` en interne, il ne faut pas que ce détail d'implémentation puisse être accédé de l'extérieur.

On peut cacher des attributs (et autres méthodes annexes) en les marquant *private* (privés).
Des attributs privés ne sont accessibles que depuis l'intérieur de la classe qui les définit.
En Python, on exprime qu'un attribut ou une méthode est privée en préfixant son nom de `__` (2 caractères `_`).
Si on tente d'accéder à ces attributs depuis l'extérieur de la classe, on obtient une erreur à l'exécution.

```python
class CashDispenser:
    __account_manager: AccountManager
    __safe: Safe

    def withdraw(self, card: BankCard, pin: int, amount: int) -> list[BankNote]:
        """..."""
        self.__account_manager.validate_pin(card, pin) # may throw ValueError
        self.__account_manager.decrease_money(card, amount) # may throw NotEnoughMoneyError
        bank_notes = self.__safe.get_notes(amount) # may throw MissingBankNoteError
        return bank_notes

cash_dispenser.withdraw(...) # ok
cash_dispenser.__safe # AttributeError: 'CashDispenser' object has no attribute '__safe'
```

Malheureusement, on ne reçoit pas d'erreur de type parce que [mypy n'implémente pas correctement les attributs privés](https://github.com/python/mypy/issues/8267).

Grâce aux attributs privés, nous avons réussi à protéger l'accès aux détails d'implémentation.
Nous avons donc correctement *encapsulé* le coffre-fort.
Ce faisant, nous continuous d'exposer la bonne *interface* au code utilisateur de la classe.

## Conclusion

Nous avons vu dans ce tutoriel le fonctionnement de base des **classes**, **attributs** et **méthodes**.
Les attributs représentent les données internes des instances des classes, tandis que les méthodes représentent les *opérations* applicables sur ces données.

Nous avons introduit les concepts **d'interface** et **d'encapsulation**.
Ces concepts sont au cœur des méthodologies de développement logiciel basées sur la programmation orientée objet.
Les méthodes, **correctement documentées**, fournissent les *interfaces*.
Les attributs **privés** permettent de mettre en œuvre *l'encapsulation*.
