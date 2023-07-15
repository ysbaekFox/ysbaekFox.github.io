---
title: "Django "
layout: archive
permalink: /categories/django
author_profile: true
sidebar:
  nav: "mainMenu"
---

{% assign posts = site.categories.django %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}