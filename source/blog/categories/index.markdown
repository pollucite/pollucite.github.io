---
layout: page
title: "categories"
date: 2014-03-13 09:54
comments: true
sharing: true
footer: true
---
<ul>

{% for item in site.categories %}

    <li><a href="/blog/categories/{{ item[0] }}/">{{ item[0] | capitalize }}</a> [ {{ item[1].size }} ]</li>

{% endfor %}

</ul>
