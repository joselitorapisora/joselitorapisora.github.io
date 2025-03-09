---
layout: home
---
# My Projects

{% for project in site.projects %}
  - [{{ project.title }}]({{ project.url }}) - {{ project.date | date: "%B %d, %Y" }}
    {{ project.content | truncate: 100 }}
{% endfor %}