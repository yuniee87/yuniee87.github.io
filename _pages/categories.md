---
layout: categories
permalink: /categories/
title: Categories
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.categories %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}