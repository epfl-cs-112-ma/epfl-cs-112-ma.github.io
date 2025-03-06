---
title: Home
layout: default
use_mermaid: true
---

# CS-112(j) Programmation Orientée Objet

Ou : Développement Logiciel

Cours *ex cathedra* : **{{ site.courseinfo.lecture_time }}, {{ site.courseinfo.lecture_room }}**

Exercices et avancement du projet : **{{ site.courseinfo.exercises_time }}, {{ site.courseinfo.exercises_rooms }}**

## Présentation générale du cours (à lire absolument)

[Présentation générale du cours](./presentation.html)

Évaluations :

| Évaluation         | Date                     | Pourcentage |
|--------------------|--------------------------|-------------|
| Midterm            | 17 avril, 9h15-11h       | 17 %        |
| Final              | 22 mai, 9h15-11h         | 33 %        |
| Projet, par binôme | 30 mai, 23h58 (deadline) | 50 %        |

⚠️ La note du projet est **minorée** à 1,5x la note des deux examens.
Par exemple, si vous obtenez 20/50 sur la somme de vos deux examens, votre de note de projet ne peut pas aller plus haut que 30/50, même s'il est parfait.

## Code utilisé pendant les cours

Vous retrouverez les bouts de code écrits chaque semaine pendant les cours [sur le repo `lectures`](https://github.com/epfl-cs-112-ma/lectures).

{% for week in site.data.weeks %}
{% assign week_nb = week.week %}
## Semaine {{ week_nb }}

Commençant le lundi {{ week.start }}.

<ol>
  {% for section in site.data.structure %}
    {% assign relevant_items = section.items | where: 'week', week_nb %}
    {% if relevant_items.size > 0 %}
      <li>
        {{ section.title }}
        <ol>
          {% for item in relevant_items %}
            <li><a href="{{ item.url }}">{{ item.title }}</a></li>
          {% endfor %}
        </ol>
      </li>
    {% endif %}
  {% endfor %}
</ol>
{% endfor %}
