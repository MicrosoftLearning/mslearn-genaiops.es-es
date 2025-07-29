---
title: Ejercicios de GenAIOps
permalink: index.html
layout: home
---

# Operacionalización de aplicaciones de IA generativa

Los siguientes ejercicios de inicio rápido están diseñados para proporcionarte una experiencia práctica de aprendizaje en la que explorarás las tareas comunes necesarias para poner en funcionamiento una carga de trabajo de IA generativa en Microsoft Azure.

> **Nota**: para completar los ejercicios, necesitarás una suscripción a Azure en la que tengas permisos y cuota suficientes para aprovisionar los recursos de Azure y modelos de IA generativa necesarios. Si aún no tienes una, puedes registrarte para obtener una [cuenta de Azure](https://azure.microsoft.com/free). Hay una opción de evaluación gratuita para los nuevos usuarios que incluye créditos durante los primeros 30 días.

## Ejercicios de inicio rápido

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% para la actividad en laboratorios %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

> **Nota**: aunque puedes completar estos ejercicios por tu cuenta, han sido diseñados para complementar módulos en [Microsoft Learn](https://learn.microsoft.com/training/paths/operationalize-gen-ai-apps/) en los que encontrarás un análisis más profundo de algunos de los conceptos subyacentes en los que se basan estos ejercicios.
