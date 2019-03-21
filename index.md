---
layout: default
title: Home
---

<section>
    <ul class="list-group">
        {% for post in site.posts %}
            <li class="post-list list-group-item">
                <a class="post-font" href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{post.title}}</a>
                <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
            </li>
        {% endfor %}
    </ul>
</section>