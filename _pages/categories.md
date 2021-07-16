---
layout: page
permalink: /categories/
title: Blog Archive
---


{: #top }

<!-- this code si from https://github.com/daattali/daattali.github.io/blob/master/index.html --> 
<div class="list-filters post-preview" style="text-align:center;font-family:Helvetica;">
  <a href="/categories" class="list-filter"> All posts </a> &nbsp;
  <a href="/CloudCategories" class="list-filter filter-selected"> Catergories </a> &nbsp;
  <a href="/CloudTags" class="list-filter"> Tags </a> &nbsp;
  <a href="/CloudDate" class="list-filter"> By Date </a> &nbsp;
</div>

<div id="archives">
{% for category in site.categories %}
  <div class="archive-group">
    {% capture category_name %}{{ category | first }}{% endcapture %}
    <div id="#{{ category_name | slugize }}"></div>
    <p></p>
    
    <h3 class="category-head">{{ category_name }}</h3>
    <a name="{{ category_name | slugize }}"></a>
    {% for post in site.categories[category_name] %}
    <article class="archive-item">
      <h4><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></h4>
    </article>
    {% endfor %}
  </div>
{% endfor %}
</div>