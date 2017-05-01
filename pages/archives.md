---
layout: page
title: Archives
keywords: 存档
comments: true
permalink: /archives/
---
<style>
@media screen and (max-width: 770px) {
    ul {
		padding-left: 20px;
		>h2 {
			// font-size: 20px;

			// margin: 0;
			margin-left: -20px;
		}
		li {
			margin: 20px 0;
			time {
				display: block;
				width: auto;
			}
			.title {
				display: block;
				// font-size: 16px;
			}
			.categories {
				// font-size: 12px;
				padding-left: 0;
				padding-right: 10px;
			}
		}
	}
}
</style>
<ul>
  {% for post in site.posts %}
	{% unless post.next %}
	  <h2 id="y{{ post.date | date: '%Y' }}">{{ post.date | date: '%Y' }}</h2>
	{% else %}
	  {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
	  {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %}
	  {% if year != nyear %}
		<h2 id="y{{ post.date | date: '%Y' }}">{{ post.date | date: '%Y' }}</h2>
	  {% endif %}
	{% endunless %}

	<li>
		<time style="display:inline-block; width: 135px;">
		{{ post.date | date:"%F" }} {{ post.date | date: "%a" }}.
		</time>
		
		<a class="title" href="{{ post.url }}">{{ post.title }}</a>
		
		<span style="padding-left: 15px"> </span>
		
		<div class="categories">
			{% for cat in post.categories %}
			<span class="meta-info">
			  <span class="octicon octicon-file-directory"></span>
			  <a href="/categories/#{{ cat }}" title="{{ cat }}">{{ cat }}</a>
			</span>
			{% endfor %}
		</div>
	</li>

  {% endfor %}
</ul>