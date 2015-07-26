---
layout: page
post_class: post-template page-template

title: Topics
permalink: /topics/
---

{% for category in site.categories %}
    {% assign cat = category | first %}
{{ cat | capitalize }}
---
{% for posts in cat %}
{% for post in posts %}
[ {{ post.title }} ] ({{ post.url }})
{% endfor %}
{% endfor %}

{% endfor %}
