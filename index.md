---
layout: page
title: Home
tagline: Just another programming blog.
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
  <!-- <div>
        post.content | postmorefilter: post.url, "Read the rest of this entry"
    </div> -->
  </li>
  {% endfor %}
</ul>
