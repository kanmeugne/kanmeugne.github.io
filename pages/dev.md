---
layout: page
show_meta: false
title: "Multi-Agent Systems"
subheadline: "Catalog"
header:
   image_fullwidth: "header_homepage_13.jpg"
permalink: "/mas/"
---
<ul>
    {% for post in site.categories["software development"] %}
    <li><a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>