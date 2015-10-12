---
layout: page
title: 每日一问
permalink: /a-question-a-day/
---

<div class="aquestionadayposts">
  {% for post in paginator.posts %}
    <article class="aquestionadayposts">

      <h1><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h1>
	  
      <div class="entry">
        {{ post.excerpt }}
      </div>

      <a href="{{ site.baseurl }}{{ post.url }}" class="read-more">Read More</a>
    </article>
  {% endfor %}
  
</div>

<br/>

<hr/>

{% if paginator.total_pages > 1 %}
    {% if paginator.previous_page %}
        <a href="{{ paginator.previous_page_path }}" style="font-weight: bold" >上一页&nbsp;&nbsp;</a>
    {% endif %}
    {% if paginator.next_page %}
        <a href="{{ paginator.next_page_path }}" style="font-weight: bold">&nbsp;&nbsp;下一页</a>
    {% endif %}
{% endif %}