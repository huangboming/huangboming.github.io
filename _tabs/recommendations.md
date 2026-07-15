---
title: 推荐阅读
icon: fas fa-book-open
order: 4
permalink: /recommended-reading/
toc: false
comments: false
description: 推荐的书籍、文章和视频
---

<p class="recommendations-intro">这里收录我认为值得一读或一看的书籍、文章和视频。（其实就是懒得单独写一篇文章来推荐 XD）</p>

{% assign recommendations = site.data.recommendations %}

{% if recommendations == empty %}

<p class="recommendations-empty">目前只是搭了个壳子，后面再慢慢加入内容。</p>

{% else %}

{% assign recommendation_days = recommendations | group_by: 'added_at' | sort: 'name' | reverse %}

<div id="recommendations">
  {% for recommendation_day in recommendation_days %}
    {% for recommendation in recommendation_day.items %}
      {% include recommendation.html recommendation=recommendation %}
    {% endfor %}
  {% endfor %}
</div>

{% endif %}
