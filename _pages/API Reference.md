---
title: "API Reference"
layout: archive
permalink: /api-reference/
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.api-reference %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}