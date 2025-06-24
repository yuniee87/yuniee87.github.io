---
title: "Japanese"
layout: archive
permalink: /japanese/
author_profile: true
---

{% assign posts = site.categories.japanese %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}