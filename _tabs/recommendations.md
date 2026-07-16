---
title: 推荐阅读
icon: fas fa-book-open
order: 4
permalink: /recommended-reading/
toc: false
comments: false
description: 推荐的书籍、文章和视频
---

<p class="recommendations-intro">收录一些我认为值得一读或一看的书籍、文章和视频。（<del>其实就是懒得写成一篇推荐文</del>）</p>

{% assign recommendations = site.data.recommendations %}

{% if recommendations == empty %}

<p class="recommendations-empty">目前只是搭了个壳子，后面再慢慢加入内容。</p>

{% else %}

{% assign recommendation_days = recommendations | group_by: 'added_at' | sort: 'name' | reverse %}

<div id="recommendations">
  {% for recommendation_day in recommendation_days %}
    <section class="recommendation-day" aria-labelledby="recommendations-{{ recommendation_day.name | escape }}">
      <h2 class="recommendation-date" id="recommendations-{{ recommendation_day.name | escape }}">
        <time datetime="{{ recommendation_day.name | escape }}">
          {{ recommendation_day.name | date: '%Y-%m-%d' }}
        </time>
      </h2>
      <div class="recommendation-list">
        {% for recommendation in recommendation_day.items %}
          {% include recommendation.html recommendation=recommendation %}
        {% endfor %}
      </div>
    </section>
  {% endfor %}
</div>

{% endif %}
