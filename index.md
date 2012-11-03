---
layout: page
title: Home
tagline: Just another programming blog.
---
{% include JB/setup %}

<article role="article">
<div id="blog-archives">

{% for post in site.posts %}
<article>

<h1 class="entry-title">
<a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
</h1>

<footer>
<time datetime="{{ post.date }}" pubdate data-updated="true">{{ post.date | date_to_string }}</time>
<span class="categories">{{ post.date | date_to_string }}</span>
</footer>

</article>
{% endfor %}

</div>
</article>
