---
title: Typage dynamique
layout: default
use_mathjax: true
---

# Typage dynamique

Vous savez déjà comment utiliser `match` pour des cas simples : tester les cas d'un `Enum`, ou tester si l'argument de `__eq__` est de votre classe, etc.
Dans ce chapitre, nous voyons plus en détail ce que peut faire `match`.
En général, cette instruction est capable de faire du *pattern matching* (reconnaissance de motifs), ce qui lui donne son nom.

En accompagnement, nous verrons quelques techniques liées au typage :

* Le test de type "à la main" avec `isinstance`
* Le transtypage avec `cast`
* Le type dynamique `Any`

⚠️ Le transtypage et le type dynamique `Any` sont *dangereux*.
Par nature, ils permettent de dire à mypy d'ignorer les règles, et de vous faire confiance à la place.
Ce sont des techniques avancées à n'utiliser qu'en dernier recours.
D'une certaine manière, à chaque fois qu'on les utilise, on a "échoué" : on n'a pas réussi à démontrer à mypy que notre code est correct.
C'est similaire à, dans une preuve mathématique, invoquer un argument du genre "je ne peux pas le prouver, mais je suis convaincu-e que $P$ est vraie, alors faites-moi confiance et laissez-moi continuer le reste de ma preuve".
Évidemment, humains faillibles que nous sommes, on se trompe souvent quand on dit ça !

## Sommaire

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}
