---
layout: page
title: Yabage
tagline: "Yet Another Blog About Geek Experiments"
---
{% include JB/setup %}

{% for post in site.posts %}
{{ post.date | date_to_string }}
##[{{ post.title }}]({{ post.url }})

{{ post.content | strip_html | truncatewords:50 }} 
[Read more]({{ post.url }})

{% endfor %}


<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

