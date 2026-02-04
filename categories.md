---
layout: page
title: Categories
permalink: /categories/
---

{% for category in site.categories %}
<h2 id="{{ category[0] | slugify }}">{{ category[0] | capitalize }}</h2>
<ul>
  {% for post in category[1] %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="post-meta">— {{ post.date | date: "%b %-d, %Y" }}</span>
  </li>
  {% endfor %}
</ul>
{% endfor %}
