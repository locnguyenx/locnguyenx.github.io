---
layout: page
title: All tags list
desc: List all tags of this site. The page with posts by tag will be generated automatically as hierachy dirs.
permalink: /tag/
---
<section id="tags">
    <ul>  
        {% capture tags %}
        {% for tag in site.tags %}
            {{ tag[0] }}
        {% endfor %}
        {% endcapture %}
        {% assign sortedtags = tags | split:' ' | sort %}

        {% for tagName in sortedtags %}            
        <li>
                <a href="{{site.baseurl}}/tag/{{ tagName }}">#{{ tagName }}</a>
        </li>
        {% endfor %}
        
    </ul>
</section>    



