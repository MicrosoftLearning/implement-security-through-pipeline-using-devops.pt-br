---
title: Implementar a segurança por meio de um pipeline usando os exercícios do Azure DevOps
permalink: index.html
layout: home
---

# Implementar a segurança por meio de um pipeline usando os exercícios do Azure DevOps

Os exercícios a seguir foram projetados para oferecer suporte aos módulos para [Implementar segurança por meio de um pipeline usando o DevOps](https://learn.microsoft.com/training/paths/implement-security-through-pipeline-using-devops/).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
