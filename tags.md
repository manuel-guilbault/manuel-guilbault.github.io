---
layout: page
title: Tags
---
{% for tag in site.tags %}
  <a href="#{{ tag[0] | slugify }}" class="tag">
    {{ tag | first }}&nbsp;
    <span class="badge">{{ tag | last | size}}</span>
  </a>
{% endfor %}

<hr/>

<ul class="posts">
  {% for tag in site.tags %}
    <h2 id="{{ tag[0] | slugify }}">{{ tag[0] | capitalize }}</h2>

    {% for post in tag[1] %}
      <li itemscope>
        <h3>
          <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
          <p class="post-date"><span><i class="fa fa-calendar" aria-hidden="true"></i> {{ post.date | date: "%B %-d" }} - <i class="fa fa-clock-o" aria-hidden="true"></i> {% include read-time.html %}</span></p>
        </h3>
      </li>
    {% endfor %}
  {% endfor %}
</ul>