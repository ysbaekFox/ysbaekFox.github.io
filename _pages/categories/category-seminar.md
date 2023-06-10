---
title: "CppCon"
layout: archive
permalink: /categories/cppcon
author_profile: true
sidebar:
  nav: "mainMenu"
---

{% assign posts = site.categories.cppcon %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}