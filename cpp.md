---
layout: page
permalink: /cpp/
title: <img src='/images/cpp.png' title='C/C++'/>
image:
categery: cpp
---

<ul class="post-list">
{% for post in site.categories.cpp %} 
  <li>
    <article>
      <a href="{{ site.url }}{{ post.url }}">
        {{ post.title }} 
        <span class="entry-date">
          <time datetime="{{ post.date | date_to_xmlschema }}">
            {{ post.date | date: "%B %d, %Y" }}
          </time>
        </span>
      </a>
    </article>
  </li>
{% endfor %}
</ul>
