---
title: "Japanese"
layout: archive
permalink: /japanese/
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.japanese %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}