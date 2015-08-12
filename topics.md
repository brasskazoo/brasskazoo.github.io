---
layout: page

title: brasskazoo - topics
permalink: /topics/
---

# Topics

<div class="topic-links">
{% for category in site.categories %} {% assign cat = category | first %} <a href="#{{ cat }}" class="button">{{ cat }} ({{site.categories[cat] | size}})</a>{% endfor %}
</div>

<br/>
{% for category in site.categories %}
    {% assign cat = category | first %}

<h3 id="{{cat}}">{{ cat | replace:'-',' ' | capitalize }}</h3>

{% for post in site.categories[cat] | sort: 'date' %}
`{{post.date  | date: "%b %d, %Y" }}` [ {{ post.title }} ]({{ post.url }})
{% endfor %}

{% endfor %}
