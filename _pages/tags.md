---
layout: tag
permalink: /tags/
title: Tags
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.tag %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}