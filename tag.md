---
layout: page
title: Tag
permalink: /tag/
---

{% for tag in site.tags %}
    <a href="/show_by_tag.html?tag={{ tag[0] }}"> {{ tag[0] }}</a>
{% endfor %}