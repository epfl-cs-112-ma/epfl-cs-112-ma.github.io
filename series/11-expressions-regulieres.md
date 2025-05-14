---
title: "Série 11 : Expressions régulières"
layout: default
---

# Série 11 : Expressions régulières

[Solutions](https://github.com/epfl-cs-112-ma/solutions-serie-11)

Cette série a pour objectif de pratiquer l'usage des expressions régulières.

Avant de commencer cette série, il est prévu que vous ayez suivi le tutoriel de cette semaine :

* [Expressions régulières](/tutoriels/expressions-regulieres.html)

Nous vous recommandons de créer [un nouveau projet Python](/references/quick-projet-setup.html) pour chaque série d'exercices.
Cela vous permettra d'isoler plus facilement le contenu des différentes semaines.

Cette série contient deux exercices :

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Reconnaissance

Pour chacune des chaînes et expressions régulières ci-dessous, identifiez si la chaîne correspond à la regex ou non.
On suppose une correspondance avec `fullmatch`.

Faites le test à la main.
Demander la réponse à Python ne sert à rien pour cet exercice.

| regex            | chaîne    |
|-----------------:|-----------|
| `arbre`          | `arbre`   |
| `arbre`          | `arbres`  |
| `a[br]+e`        | `arbre`   |
| `a[br]+e`        | `are`     |
| `a[br]+e`        | `ae`      |
| `a[br]*e`        | `ae`      |
| `[aeiouy]`       | `u`       |
| `[aeiouy]`       | `b`       |
| `[aeiouy]+`      | `oui`     |
| `[aeiouy]+`      | `non`     |
| `ab|cd`          | `cd`      |
| `ab|cd`          | `abd`     |
| `a(b|c)d`        | `abd`     |
| `ab|cd?`         | `abc`     |
| `ab|cd?`         | `c`       |
| `ab|cd?`         | `ab`      |
| `ab|cd?`         |           |
| `(ab|cd)?`       |           |
| `bonjour|soir`   | `bonjour` |
| `bonjour|soir`   | `bonsoir` |
| `bon(jour|soir)` | `bonjour` |
| `bon(jour|soir)` | `bonsoir` |
| `bon(jour|soir)` | `bon`     |
| `x|y`            | `x|y`     |
| `x\|y`           | `x|y`     |
| `a[f-m]+z`       | `aghlfz`  |
| `a[f-m]+z`       | `aghzfz`  |

## Traducteur de dates

Dans la francophonie européenne, on utilise normalement le format de date `31/12/2025`.
Aux États-Unis, on utilise plutôt `12-31-2025`.
En Hongrie encore, on utilise le format `2025-12-31`.

Écrivez une fonction qui accepte un texte quelconque et traduit les dates du format états-unien vers le format francophone.

Puis, généralisez votre fonction pour accepter le format de destination en paramètre supplémentaire (une énumération).
Cette version généralisée doit traduire toutes les dates, quel que soit leur format, dans le format cible.
Tentez d'utiliser une seule expression régulière pour faire cela.
