---
layout: page
post_class: post-template page-template

title: Topics
permalink: /topics/
---

{% for category in site.categories %}
    {% assign cat = category | first %}
{{ cat }} ({{site.categories[cat] | size}})
---
{% for post in site.categories[cat] | sort: 'date' %}
`{{post.date  | date: "%b %d, %Y" }}` [ {{ post.title }} ]({{ post.url }})
{% endfor %}

{% endfor %}
