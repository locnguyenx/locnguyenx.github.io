---
layout: page
title: Posts by category
desc: Find posts by category here
permalink: /categories/
adallow: 0
---

<!-- main content of category-->
<section id="categories">
    {% assign sorted_cats = site.categories | sort %}
    {% for category in sorted_cats %}
    {% assign sorted_posts = category[1] | reversed %}
    <h2 id="{{category[0] | uri_escape | downcase | slugify }}">{{category[0] | camelcase }}</h2>
    <ul class="post-list">
        {% for post in sorted_posts %}
        {% unless post.draft %}

        {% if post.menutitle %}
        {% assign title = post.menutitle %}
        {% else %}
        {% assign title = post.title %}
        {% endif %}

        <li>
            <div class="article">
                <article class="article" itemscope itemtype="http://schema.org/BlogPosting">
                    <header class="post-header">
                        <span class="title"><a itemprop="name" href="{{ post.url | prepend: site.github.url | prepend: site.baseurl }}" title="{{ title }}">{{ title }}</a></span>
                      
                    </header>
                </article>
            </div>
        </li>
        {% endunless %}
        {% endfor %}
    </ul>
    {% endfor %}
</section>
