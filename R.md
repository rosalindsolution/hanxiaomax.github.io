---
layout: page
permalink: /R/
title: <img src="http://i.imgur.com/IUX1VUH.png" title="Hosted by imgur.com"/>

---

<ul class="post-list">
{% for post in site.categories.matlab %} 
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
