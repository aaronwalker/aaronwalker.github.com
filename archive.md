---
layout: default
title: aaronwalker.me - archive
---

<div id="posts">
	<ul>
	{% for post in site.posts %}
	<li>
		<h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
		<div class="posted">Posted on {{ post.date | date_to_string }}</div>
		<br/>
		<hr/>
	</li>
	{% endfor %}
	</ul>
</div>