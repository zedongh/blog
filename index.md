---
layout: default
title: Home
---

<section>
    <ul class="list-group">
        {% for post in site.posts %}
            <li class="post-list list-group-item">
                <a class="post-font" href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a>
                <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
                <span>
                    <ul class="title-categories">
                        {% for cat in post.categories %}
                            <li class="title-category">{{cat}}</li>
                        {% endfor %}
                    </ul>
                </span>
            </li>
        {% endfor %}
    </ul>
</section>