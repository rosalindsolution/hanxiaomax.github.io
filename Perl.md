---
layout: page
permalink: /Perl/
title: <img src='/images/perl.png' title='Perl'/>
image:
categery: Perl
---

<ul class="post-list">
{% for post in site.categories.c51%} 
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
