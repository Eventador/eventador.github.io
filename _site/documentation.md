{% for doc in site.documentation limit:3 %}
      <li>
        <a href="{{ doc.url }}">{{ doc.title }}</a>
        <p>{{ doc.short-description }}</p>
      </li>
{% endfor %}
