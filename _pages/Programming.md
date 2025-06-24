---
title: "Programming"
layout: archive
permalink: /programming/
author_profile: true
---

{% assign posts = site.categories.programming %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}