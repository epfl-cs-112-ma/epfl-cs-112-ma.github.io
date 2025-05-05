---
title: Expressions régulières
layout: default
use_mathjax: true
---

# Expressions régulières

Les expressions régulières (*regular expressions*, souvent abrégées en *regex* ou *regexp*) sont un moyen concis et efficace d'analyser des chaînes de caractères qui suivent un *motif* particulier.

Chaque langage qui se respecte possède une bibliothèque de regexp.
Souvent critiquées pour être obscures, elles n'en restent pas moins très utiles.
Les regexp doivent faire partie de la boîte à outil de tout développeur ou développeuse.

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Utilisation

Commençons par voir comment utiliser le module d'expressions régulières de Python : [`re`](https://docs.python.org/3/library/re.html#module-re).

```python
import re

as_and_b = re.compile('a+b')

print(as_and_b.fullmatch('aaab'))  # <re.Match object>
print(as_and_b.fullmatch('ab'))    # <re.Match object>
print(as_and_b.fullmatch('b'))     # None
print(as_and_b.fullmatch('aaaba')) # None
print(as_and_b.fullmatch('aaabb')) # None
print(as_and_b.fullmatch('a+b'))   # None
```

La chaîne `a+b` est une expression régulière.
Une expression régulière représente un *motif* (*pattern*), auquel peuvent correspondre, ou pas, d'autres chaînes de caractères.

Ici, la chaîne `'aaab'` correspond au motif `a+b`, alors que `'aaaba'` ne correspond pas.
De manière générale, l'expression régulière `a+b` décrit le motif suivant :

* 1 ou plusieurs `a` qui se suivent ;
* puis exactement 1 `b`.

La chaîne `'aaab'` correspond effectivement à ce motif (3 `a` qui se suivent, puis un `b`), alors que `'aaaba'` contient un `a` après l'unique `b`.

L'utilisation d'une expression régulière se fait normalement en deux temps :

* On *compile* l'expression régulière en tant que telle, produisant un objet de type `re.Pattern` (c'est l'objet `as_and_b` ici) ;
* On utilise l'objet `re.Pattern` pour faire des tentatives de *correspondance* (*match*) avec des chaînes de caractères.

Notez bien l'asymétrie.
Bien que l'expression régulière soit initialement décrite elle-même comme une chaîne de caractères, celle-ci représente un motif.
D'ailleurs la chaîne `'a+b'` ne correspond *pas* au motif `a+b` (dernier exemple), puisqu'elle ne contient pas "1 à plusieurs `a` suivis d'exactement 1 `b`" : elle contient un `+` superflu.

Lorsqu'une chaîne ne correspond pas, `fullmatch` renvoie `None`.
Lorsqu'une chaîne correspond, `fullmatch` renvoie un objet de type `re.Match`, qui contient plus d'informations (nous y reviendrons).
Puisque `None` est "falsy" et que les instances de `re.Match` sont "truthy", on peut utiliser le résultat de `fullmatch` comme condition d'un `if` :

```python
if as_and_b.fullmatch('aaab'):
    print("Ceci est affiché.")

if as_and_b.fullmatch('aaabb'):
    print("Ceci n'est pas affiché.")
```

### Extraction de groupes

En plus de vérifier qu'une chaîne correspond à un motif, on veut parfois extraire les parties de la chaîne qui correspondent à des sous-motifs donnés.
Un exemple typique est d'identifier une date au format `D/M/Y`, et d'extraire les parties année, mois et jours.
On peut utiliser l'expression régulière `[0-9]+/[0-9]+/[0-9]+` pour tester, mais cela ne nous aide pas à extraire ce qui nous intéresse.
Pour cela, on ajoute des parenthèses pour former des *groupes* :

```python
date_re = re.Pattern('([0-9]+)/([0-9]+)/([0-9]+)')

result = date_re.fullmatch('1/5/2025')
if result:
    print(result.group(1)) # '1'
    print(result.group(2)) # '5'
    print(result.group(3)) # '2025'
```

La méthode `group` de `re.Match` renvoie la portion de la chaîne qui correspondait au *groupe* donné.
Les groupes sont les sous-motifs parenthésés, et sont comptés à partir de 1.

Le sous-motif `[0-9]` représente un caractère parmi les chiffres du répertoire ASCII.
Le `+` juste derrière indique qu'on peut répéter ce sous-motif de 1 à plusieurs fois.
`[0-9]+` est donc un motif qui représente de 1 à plusieurs chiffres (chaque chiffre peut être différent, comme dans `2025`).

### Recherche dans une chaîne

Plutôt que de tester si une chaîne entière correspond à un motif, on veut parfois *chercher* dans une chaîne les occurrences du motif.
Les méthodes [`search`](https://docs.python.org/3/library/re.html#re.Pattern.search), [`findall`](https://docs.python.org/3/library/re.html#re.Pattern.findall) et [`finditer`](https://docs.python.org/3/library/re.html#re.Pattern.finditer) servent à ça.
La plus utile est sans doute cette dernière.
Elle renvoie un `Iterator[re.Match]`, qui retourne un `re.Match` pour chaque occurrence du motif dans la chaîne.
Le groupe implicite 0 permet alors de récupérer le morceau de la chaîne qui correspondait (en plus des sous-groupes explicites) :

```python
txt = """
La Confédération suisse, telle qu'on la connaît, a reçu sa constitution
le 12/9/1848. Elle a rejoint l'ONU le 11/9/2002. Mais comme elle a été
reconnue neutre le 20/5/1815 perpétuellement, ce n'est pas demain la
veille qu'on la verra dans l'OTAN.
"""

for date_match in date_re.finditer(txt):
    print(date_match.group(0))
# 12/9/1848
# 11/9/2002
# 20/5/1815
```

### Autres fonctionnalités

Consultez la documentation de [`re.Pattern`](https://docs.python.org/3/library/re.html#regular-expression-objects) et [`re.Match`](https://docs.python.org/3/library/re.html#match-objects) pour voir quelles autres méthodes peuvent vous être utiles.

## Syntaxe des expressions régulières

Au travers des exemples ci-dessus, vous avez pu glaner quelques éléments de syntaxe des expressions régulières :

* Une lettre comme `a` ou `b` est un motif auquel correspond exactement cette lettre dans la chaîne.
* Un intervalle de caractères entre crochets, comme `[0-9]`, correspond à *un* caractère dans l'intervalle.
* Le `+` *modifie* le sous-motif précédent pour qu'il soit répété 1 à plusieurs fois.
* Les parenthèses `()` créent des *groupes* que l'on peut ensuite extraire.

Vous avez sans doute remarqué que c'est une syntaxe très *dense*.
Il n'y a pas de place pour des espaces.
Chaque caractère compte.
C'est souvent ce pourquoi beaucoup de gens les qualifient *d'obscures*.

Cependant, vous avez l'habitude de manipuler un autre langage qui a une syntaxe très dense : le langage des mathématiques !
Sans apprentissage, vous seriez probablement totalement démuni-e face à la [formule de Stirling](https://fr.wikipedia.org/wiki/Formule_de_Stirling) :

$$ \lim_{n\to +\infty } {n\,! \over {\sqrt {2\pi n}}\;\left({n}/{\rm {e}}\right)^{n}} = 1 $$

Dans cette formule, chaque caractère compte aussi !
Pourtant, avec votre bagage mathématique, vous pouvez y donner du sens.

Cette densité est utile.
Avant l'invention du langage mathématique, les traités de mathématique et physique comportaient des phrases en français telles que

> [Loi de la gravitation universelle](https://interfas.univ-tlse2.fr/nacelles/1853) : Deux corps quelconques s'attirent en raison directe de leur masse et en raison inverse du carré de la distance de leurs centres de gravité.

que nous écririons aujourd'hui

$$ {F}_{A/B}={F}_{B/A}=G{\frac {M_{A}M_{B}}{d^{2}}} $$

Il y a de bonnes chances que vous comprenez *mieux* la formulation dense que celle "légère" en français.

Il en va de même avec les expressions régulières.
Il faut se forcer à apprendre leur syntaxe, mais une fois connue, celle-ci permet d'exprimer de manière concise et précise de nombreux motifs utiles.

Voyons maintenant les éléments de syntaxe les plus courants.

### Caractère littéral

Sauf indication contraire, un caractère $c$ se correspond à lui-même.
On dit que c'est un *littéral*.

### Ensemble de caractères

Des caractères entre crochets `[]` correspondent à exactement 1 caractère *parmi* l'ensemble.

Les caractères peuvent être individuels (`[ACGT]` correspond à l'un quelconque de ces 4 caractères) ou par *intervalle* identifié par le caractère spécial `-` (`[0-9]`).
Les intervalles de caractères sont définis par les intervalles de *code point* correspondants dans la table géante d'[Unicode](https://fr.wikipedia.org/wiki/Unicode).
Souvent, on ne se sert des intervalles que dans le sous-ensemble des 128 premiers caractères d'Unicode, connus sous le nom d'[ASCII](https://fr.wikipedia.org/wiki/American_Standard_Code_for_Information_Interchange).

On peut combiner les deux techniques : le motif `[0-9A-Za-Z_]` représente un caractère parmi a) les chiffres ASCII, b) les lettres majuscules et minuscules ASCII et c) le caractère `_`.
Les identificateurs dans nos programmes sont souvent constitués de caractères dans cet ensemble (bien que les langages modernes acceptent bien plus de caractères tirés de l'espace Unicode).

### Répétitions

Les caractères `?`, `+` et `*` sont spéciaux.
Ils *modifient* le motif qui les précède immédiatement de sorte qu'il puisse être répété dans la chaîne :

* `𝑋?` répète `𝑋` de 0 à 1 fois (c'est donc un `𝑋` optionnel, plus que répété) ;
* `𝑋*` répète `𝑋` de 0 à plusieurs fois ;
* `𝑋+` répète `𝑋` de 1 à plusieurs fois.

On peut aussi répéter exactement $n$ fois un sous-motif avec des accolades :

* `𝑋{3}` répète `𝑋` exactement 3 fois ;
* `𝑋{3,}` répète `𝑋` au moins 3 fois ;
* `𝑋{3,5}` répète `𝑋` de 3 à 5 fois.

Des variantes avec un `?` ou un `+` supplémentaires existent (comme `𝑋?+` ou `𝑋*?`) mais sont assez avancées.

### Séquences

Lorsqu'on accole deux sous-motifs, ils doivent se succéder dans la chaîne.
`ab` correspond à un `a` suivi d'un `b`.
De manière générale, une chaîne `s` correspond au motif `𝑋𝑌` si :

* on peut la décomposer en deux sous-chaînes `s1` et `s2` de sorte que `s == s1 + s2` (au sens de concaténation en Python) ;
* `s1` correspond à `𝑋` ;
* et `s2` correspond à `𝑌`.

Notez que ce "on peut la décomposer" est très puissant.
Ce n'est pas à vous de trouver la bonne façon de décomposer la chaîne.
L'expression régulière trouvera une façon, si elle est existe.

Par exemple, la chaîne `abbabca` correspond au motif `[abc]+a[abc]+` en décomposant en `abb`, `a` et `bca`.
Ce n'est pas évident !
Le premier `+` aurait pu se répéter plus ou moins que 3 fois ; mais tous les autres choix auraient donné une décomposition qui ne fonctionne pas *au total*.
Par exemple, si `[abc]+` avait été associé à `abbabc` dans la chaîne, le `a` suivant aurait correspondu au dernier `a` de la chaîne, mais le second `[abc]+` n'aurait plus rien eu à s'attribuer.

Priorité : les opérateurs de répétition ont *priorité* sur la séquence.
`ab+` représente bien 1 `a` suivi de 1 ou plusieurs `b` (et non de 1 à plusieurs fois la séquence `ab`).

### Alternatives

L'opérateur `|` sépare des alternatives.
`a|bc` représente soit l'unique caractère `a`, soit la séquence `bc`.
De manière générale, une chaîne correspond au motif `𝑋|𝑌` si elle correspond *soit* au motif `𝑋`, soit au motif `𝑌`.

(Elle peut aussi correspondre aux deux, auquel cas `𝑋` est privilégié pour les questions de groupes.)

Priorité : la séquence (et *a fortiori* les opérateurs de répétition) on priorité sur les opérateurs d'alternatives.

Notez que les ensembles de caractères comme `[abc]` peuvent s'exprimer sous forme d'alternatives `a|b|c` (mais pas avec la même priorité).

### Groupes

Les parenthèses `()` ont deux fonctions : créer des *groupes* que l'on peut extraire, mais aussi modifier la priorité des opérateurs (comme en mathématiques).
Le motif `(a|bc)d` représente donc soit la chaîne `ad`, soit la chaîne `bcd` (mais pas la chaîne `a`).

Si vous voulez des parenthèses *uniquement* pour la faculté de modifier la priorité, vous pouvez utiliser un *groupe non-capturant* `(?:𝑋)`.
Les `?:` sont de la syntaxe spéciale dans ce contexte, qui annule le caractère de *groupe* mais garde la priorité.

### Échappement

Si vous devez reconnaître de manière littérale un caractère qui serait un caractère spécial de la syntaxe, vous devez l'échapper avec un `\`.
Par exemple, le motif `\|` correspond au caractère `|` dans la chaîne (plutôt que de séparer des alternatives).

Le caractère `\` étant lui-même spécial, il faut donc aussi l'échapper avec `\\`.

⚠️ `\` étant spécial au niveau de la *syntaxe Python des chaînes de caractères*, il faut systématiquement le dédoubler, comme dans `re.compile('\\|')` pour faire un motif qui reconnaît exactement le caractère `|`.
Cela rend les expressions régulières absolument méconnaissables.
Il vaut mieux dans ce cas utiliser une [raw string](https://docs.python.org/3/reference/lexical_analysis.html#string-and-bytes-literals) en la préfixant du caractère `r` : `re.compile(r'\|')`.
Le `r` empêche Python de traiter les `\` de manière spéciale, ce qui vous laisse les utiliser pour leur sens de syntaxe d'expression régulière.

### `^`, `$` et `.` (point)

Je mentionne ces caractères pour que vous sachiez qu'ils sont spéciaux.

⚠️ Ils sont presque systématiquement utilisés à mauvais escient.

La quasi-totalité des tutoriels ou des réponses d'IA sur les expressions régulières vous montreront `^` et `$`.
**Ils auront tord** dans 99 % des cas.
Presque tout le monde se trompe sur ce que font ces caractères.

**Non sérieusement, même moi** j'avais mal compris ce que faisait `$` jusqu'à ce que je doive *implémenter moi-même* un moteur d'expressions régulières.

L'usage de `fullmatch` (et non de `match`) vous dispensera de toute nécessité d'utiliser ces caractères.

Quant à `.`, si vous en avez besoin, assurez-vous d'utiliser le flag [`DOTALL`](https://docs.python.org/3/library/re.html#re.DOTALL) lorsque vous compilez votre expression régulière : `re.compile('a.b', re.DOTALL)`.
Avec ce flag, le `.` reconnaît exactement 1 caractère *quelconque*.

### Vous en voulez plus ?

Consultez la [documentation officielle de syntaxe des expression régulières en Python](https://docs.python.org/3/library/re.html#regular-expression-syntax).

## Exemples

Voici quelques exemples complets pour illustrer le pouvoir des expressions régulières :

* `[0-9]{4}-[0-9]{2}-[0-9]{2}` \
  date au format YYYY-MM-DD (avec exactement 4, 2 et 2 chiffres)
* `([0-9]{4})-([0-9]{2})-([0-9]{2})` \
  la même chose, mais on extrait les trois sous-groupes de chiffres
* `colou?r` \
  les deux façons d'écrire correctement le mot "couleur" en anglais
* `[0-9]+(?:\.[0-9]+)?` \
  un nombre décimal, potentiellement avec une partie fractionnaire (si le `.` est présent, il doit y avoir au moins un chiffre de chaque côté)
* `([0-9]+)(?:\.([0-9]+))?` \
  la même chose, mais on extrait les parties entière et fractionnaire dans des groupes (s'il n'y a pas de `.`, la partie fractionnaire sera `None`)
* `(?:AT|CG)+` \
  une séquence ADN bien formée
* `M{0,4}(CM|CD|D?C{0,3})(XC|XL|L?X{0,3})(IX|IV|V?I{0,3})` \
  nombre en chiffres romains ([trouvée sur Internet](https://regexr.com/3a406), je ne l'ai pas testée à fond)

## Éditeur en ligne

Il est utile de pouvoir éditer et tester une expression régulière en dehors du programme pour la mettre au point.
Le site [Pythex](https://pythex.org/) vous propose ce service avec le format reconnu par Python.

Cela ne vous dispense pas d'écrire les tests nécessaires à votre application une fois que vous avez conçu votre expression régulière.

## Un petit jeu ?

Vous pouvez entraîner votre regex-fu avec le jeu en ligne [regex crossword](https://regexcrossword.com/) (mots-croisés d'expressions régulières).
Attention, c'est addictif. :-)
