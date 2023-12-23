---
title: "Manual"
layout: archive
permalink: /manual
---


{% assign posts = site.categories.manual%}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}