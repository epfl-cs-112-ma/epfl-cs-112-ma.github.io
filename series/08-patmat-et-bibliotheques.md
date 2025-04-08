---
title: "Série 8 : Pattern matching et bibliothèques"
layout: default
use_mathjax: true
use_mermaid: true
---

# Série 8 : Pattern matching et bibliothèques

[Solutions](https://github.com/epfl-cs-112-ma/solutions-serie-08)

Cette série a pour objectif de pratiquer l'usage de bibliothèques tierces, ainsi que le pattern matching.

Avant de commencer cette série, il est prévu que vous ayez suivi les deux premiers tutoriels de cette semaine :

* [Pattern matching](/tutoriels/pattern-matching.html)
* [Bibliothèques](/tutoriels/bibliotheques.html)

Il n'est pas nécessaire d'avoir suivi le tutoriel sur les [fonctions variadiques](/tutoriels/varargs.html), bien que cela peut donner un meilleur éclairage sur le pattern matching des collections.

Nous vous recommandons de créer [un nouveau projet Python](/references/quick-projet-setup.html) pour chaque série d'exercices.
Cela vous permettra d'isoler plus facilement le contenu des différentes semaines.

⚠️ Ceci est particulièrement importement pour cette série, puisque le but est justement d'utiliser des bibliothèques externes, qui doivent être isolées par projet.

Cette série contient un unique exercice :

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Requête HTTP et JSON

> Cet "exercice" se rapproche plus d'un "tutoriel étendu".
> Il sera beaucoup plus guidé que d'habitude, et introduit même certains concepts concernant les APIs HTTP et le format JSON.

**Remarque générale** : écrire des tests pour cet exercice sort du cadre de ce cours.
Exceptionnellement, on ne vous le demande pas du tout.

Pour enraciner les concepts de cette semaine, nous allons développer une mini-application.
Celle-ci nous dira quelle est la version la plus récente d'une bibliothèque disponible sur PyPI.

Pour cela, nous allons devoir utiliser *l'API* (*Application Programming Interface*) de PyPI.
Dans ce contexte, une API est un ensemble de services mis à disposition par un site Web à l'usage de programmes informatiques.
Cela s'oppose à *User Interface* (UI), qui peut contenir les mêmes services, mais à destination d'humains.

On va donc commencer par se documenter sur l'API de PyPI.
Une recherche telle que "PyPI API" nous amène à la [documentation officielle](https://docs.pypi.org/api/).
Après quelques mises en gardes qui ne nous concernent pas (parce qu'on ne va pas faire des centaines de requêtes), nous arrivons à la section qui nous intéresse, [Official guidance](https://docs.pypi.org/api/#official-guidance), qui nous envoie à [la référence sur l'API JSON](https://docs.pypi.org/api/json/).

> JSON veut dire "JavaScript Object Notation".
> Là où HTML est le format standard pour que les sites Web communiquent avec des humains, le format JSON est le plus populaire pour communiquer avec des programmes.
> Nous allons voir tout de suite à quoi il ressemble.

La référence commence par la requête à utiliser pour "Get a projet", donc pour obtenir les informations sur une bibliothèque lorsque l'on connaît son nom.
Bingo, c'est justement ce dont nous avons besoin pour notre petite application.
Elle nous dit que la requête à effecteur est

```
GET /pypi/<project>/json
```

La partie `<project>`, entre chevrons, est un *paramètre*.
On la remplacera par le nom de la bibliothèque, pour donner par exemple `/pypi/arcade/json`.
Ceci viendra s'ajouter après le nom de domaine de PyPI, pour un total de `https://pypi.org/pypi/arcade/json`.

Le "GET" est le type de requête HTTP.
Rappelez-vous (cours ICC) que HTTP est un protocole réseau de niveau 5 (Application) pour communiquer avec un site Web.
Les requêtes dans ce protocole comportent toujours une *méthode* (GET ici) et une *URL* (`https://pypi.org/pypi/arcade/json` dans notre cas).
La méthode GET est celle par défaut.

Ça nous arrange bien, car ça veut dire qu'on peut facilement "tester" cette requête dans un navigateur Web.
[Allez donc voir](https://pypi.org/pypi/arcade/json).
Votre navigateur vous présentera certainement la réponse (JSON) de manière interactive.
Naviguez dedans pour essayer de trouver ce qui nous intéresse, à savoir la dernière version en date.

Vous voir un exemple de la réponse au format texte (non interactif, et donc tel que la verra votre programme) dans la référence.

Il peut aussi être utile, lorsqu'on teste des APIs de ce type, d'essayer les requêtes dans un shell.
Au lieu de demander à un navigateur, vous demandez à un outil en ligne de commande, qui vous montrera le texte brut reçu en réponse.
Vous pouvez faire des requêtes avec le logiciel `curl`, par exemple.
Sur Linux et MacOS, il devrait être disponible par défaut.
Sur Windows, il sera normalement disponible dans votre `git bash`, mais pas forcément dans PowerShell ou dans la ligne de commande standard.

Vous pouvez essayer d'accéder à l'URL ci-dessus avec curl :

```bash
$ curl https://pypi.org/pypi/arcade/json
```

Qui va répondre au format texte brut de JSON.
Le résultat est très dense et difficilement lisible humainement.
Cependant, vous reconnaîtrez peut-être ce qui semblent être des listes `[...]` et des dictionnaires `{"k":v, ...}`.
Bien que ce soit une syntaxe originaire de JavaScript et non de Python, la ressemblance est frappante.

### Faire la requête depuis Python

Vous allez maintenant devoir faire cette même requête, mais depuis Python, afin de récupérer le résultat dans une `str`.
Plus tard, vous devrez analyser cette string pour en récupérer l'information utile.

Une question se pose : comment faire une requête HTTP avec Python ?
Une petite recherche sur Internet nous amène deux résultats dominants :

* La documentation officielle de Python sur un module appelé [`http.client`](https://docs.python.org/3/library/http.client.html).
* Une bibliothèque externe appelée [Requests](https://requests.readthedocs.io/en/latest/).

Les modules officiels devraient recevoir la priorité.
En l'occurrence, la page de documentation de `http.client` nous dit tout de suite qu'on devrait probablement utiliser Requests :

> **See also:** The Requests package is recommended for a higher-level HTTP client interface.

Si même la documentation officielle le dit, on va plutôt se tourner vers Requests.

⚠️⚠️⚠️ Soyez *extrêment critiques* lorsque vous devez installer des bibliothèques externes.
Installer une bibliothèque externe, c'est **donner à une personne inconnue le droit d'exécuter *ce qu'elle veut* sur votre ordinateur**.
Faites votre recherche.
Vérifiez que la bibliothèque est digne de confiance.
Croisez différentes sources d'opinions sur la bonne bibliothèque à utiliser.
Ne faites pas confiance au premier résultat de votre IA favorite, pas plus qu'au premier résultat Google, Reddit, ou TikTok.
Le piratage *via* bibliothèques frauduleuses est un *réel problème de sécurité* en 2025.
⚠️⚠️⚠️ (oui j'insiste avec tous mes ⚠️)

Maintenant qu'on a choisi Requests, consultez sa documentation (pas *tout*, évidemment).
Vous aurez besoin des informations suivantes :

* Comment l'installer (rappel : on vous donnera souvent `pip install`, mais vous devez utiliser `uv add` à la place) ;
* Comment faire une requête simple (HTTP GET) ;
* Comment vérifier qu'elle s'est bien déroulée (le "status code" devrait être [200 OK](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/200)) ;
* Comment obtenir le résultat.

Après l'avoir installée, vous devrez pouvoir faire l'import suivant :

```python
import requests
```

Cependant, mypy se plaindra avec le message suivant :

```
Library stubs not installed for "requests"
Hint: "python3 -m pip install types-requests"
(or run "mypy --install-types" to install all missing stub packages)
See https://mypy.readthedocs.io/en/stable/running_mypy.html#missing-imports
```

C'est parce que, contrairement à Arcade, Requests ne distribue pas ses types par défaut.
Heureusement, il est possible de les installer séparément *via* la bibliothèque `types-requests`, que vous devrez donc installer aussi.

**Implémentez** une fonction `main` qui fait la requête HTTP mentionnée plus haut, puis affiche son résultat brut (texte) dans le shell (un peu comme si vous réimplémentiez `curl`).

**Ajustez** votre programme pour qu'on puisse personnaliser la bibliothèque pour laquelle on veut le résultat.
Au lieu que ce soit toujours `arcade`, on veut pouvoir passer le nom de la bibliothèque en argument à `uv run main.py`, comme par exemple

```bash
$ uv run main.py lichess
```

### Interpréter le JSON

Au lieu d'afficher le texte brut, on veut extraire la dernière version en date, et afficher uniquement ce numéro de version sur le shell.

Utilisez à nouveau la documentation de Requests pour savoir comment intepréter la réponse comme du JSON.

**Adaptez** votre programme pour extraire le JSON, le manipuler et obtenir le bon numéro de version.

**Attention** : Requests va vous donner le résultat JSON sous la forme d'une valeur typée comme `Any`.
`Any` est un type dangereux à n'utiliser qu'en connaissance de cause.
Comme vous n'avez pas encore apris à l'utiliser, assurez-vous de toute de suite assigner le résultat à une variable de type `object` (rappel : le supertype de tous les types).

*Conseil :* utilisez le pattern matching pour manipuler la réponse est trouver le numéro de version.

Vérifiez que votre programme est résistant aux erreurs suivantes :

* Donner un nom de bibliothèque qui n'existe pas ;
* Tenter d'exécuter le programme lorsqu'il n'y a pas de connexion Internet (coupez votre Wifi pour essayer cela ; sur Linux vous pouvez aussi lancer `unshare -r -n uv run main.py` pour déconnecter uniquement ce programme d'Internet).

### Exercez-vous

Explorez la version interactive du JSON proposée par votre browser.
Identifiez d'autres informations qu'il serait intéressant de montrer.
Exercez ainsi votre pattern matching pour trouver différents types d'informations.
