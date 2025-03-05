---
title: Instructions hébergées en ligne
permalink: index.html
layout: home
---

# Répertoire de contenu

Les liens hypertexte vers chaque exercice et démonstration de labo sont répertoriés ci-dessous.

## Labos \(Entra ID\)

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs_EntraID'" %}
| Module | Laboratoire |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

## Labos \(AD DS\)

Les fichiers de labo requis peuvent être [TÉLÉCHARGÉS ICI](https://github.com/MicrosoftLearning/AZ-140-Configuring-and-Operating-Microsoft-Azure-Virtual-Desktop/archive/master.zip)

{% assign labs = site.pages | where_exp:"page", "page.url contains '_ADDS'" %}
| Module | Laboratoire |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

## Labos \(Entra DS\)

Les fichiers de labo requis peuvent être [TÉLÉCHARGÉS ICI](https://github.com/MicrosoftLearning/AZ-140-Configuring-and-Operating-Microsoft-Azure-Virtual-Desktop/archive/master.zip)

{% assign labs = site.pages | where_exp:"page", "page.url contains '_AADDS'" %}
| Module | Laboratoire |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

<!--
## Demos

{% assign demos = site.pages | where_exp:"page", "page.url contains '/Instructions/Demos'" %}
| Module | Demo |
| --- | --- | 
{% for activity in demos  %}| {{ activity.demo.module }} | [{{ activity.demo.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
-->
