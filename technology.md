---
layout: page
permalink: /categories/technology
title: Technology
---

<div id="archives">
  <div class="archive-group">
    {% for post in site.categories.Technology %}
     <li><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </div>
</div>
