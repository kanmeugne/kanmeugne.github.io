---
layout: page
show_meta: false
title: "Design Articles"
subheadline: "Catalog"
header:
   image_fullwidth: "header_unsplash_5.jpg"
permalink: "/design/"
---
<ul>
    {% for post in site.categories.design %}
    <li><a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>