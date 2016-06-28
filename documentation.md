---
layout: page
title: Documentation
permalink: /documentation/
---
<div>
<hr><br>
{% for doc in site.documentation %}
      <li>
        <a href="{{ doc.url }}">{{ doc.title }}</a>
        <p>{{ doc.short-description }}</p>
      </li>
{% endfor %}
</div>
