---
layout: page
permalink: /Python/
title: <img src='/images/python.png' title='Python'/>
image:
categery: Python
---

<ul class="post-list">
{% for post in site.categories.python%} 
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
