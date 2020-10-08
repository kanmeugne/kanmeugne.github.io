---
layout: page
show_meta: false
title: "Software Development Articles"
subheadline: "Catalog"
header:
   image_fullwidth: "header_unsplash_5.jpg"
permalink: "/dev/"
---
<ul>
    {% for post in site.categories.dev %}
    <li><a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>