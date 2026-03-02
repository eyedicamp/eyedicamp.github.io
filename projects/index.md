---
layout: project
title: All Projects
excerpt: "A List of Projects"
comments: false
---

{% raw %}
## Projects

{% assign items = site.posts | where_exp: "p", "p.categories contains 'projects'" %}
{% if items.size == 0 %}
No project posts found. Make sure your project post has `categories: [projects, ...]`.
{% else %}
<ul>
  {% for p in items %}
    <li>
      <a href="{{ p.url | relative_url }}">{{ p.title }}</a>
      <small>({{ p.date | date: "%Y-%m-%d" }})</small><br/>
      {% if p.excerpt %}{{ p.excerpt }}{% endif %}
    </li>
  {% endfor %}
</ul>
{% endif %}
{% endraw %}