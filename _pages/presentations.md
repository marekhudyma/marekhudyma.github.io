---
layout: default
title: Presentations
permalink: /presentations/
---

<div class="wrapper">
  <div class="row">
    <div class="post-list">
      {% for post in site.posts %}
      {% if post.type contains 'presentation' %} 
        {% include post-card.html %}
        {% endif %}
      {% endfor %}
    </div>
  </div>
</div>