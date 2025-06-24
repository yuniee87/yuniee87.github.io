---
title: "English"
layout: archive
permalink: /english/
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.english %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}