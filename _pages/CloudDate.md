---
layout: page
title: Blog Archive With Date
permalink: /CloudDate/
---
{: #top }


<!-- this code si from https://github.com/daattali/daattali.github.io/blob/master/index.html --> 
<div class="list-filters post-preview" style="text-align:center;font-family:Helvetica;">
  <a href="/categories" class="list-filter"> All posts </a> &nbsp;
  <a href="/CloudCategories" class="list-filter filter-selected"> Catergories </a> &nbsp;
  <a href="/CloudTags" class="list-filter"> Tags </a> &nbsp;
  <a href="/CloudDate" class="list-filter"> By Date </a> &nbsp;
</div>

<div class="post-preview">
{% assign openList = '<ul class="later on">'  %}
{% assign closeList = '</ul>' %}
	{% assign archive_url = site.baseurl | append: '/CloudDate/' %}
	{% for post in site.posts %}
		{% assign currentdate = post.date | date: '%B %Y' %}
			{% if currentdate != date %}
				{% unless forloop.first %}
					<i class="badge">{{ count }}</i></span></a>
				{% endunless %}
				{% assign count = 1 %}
				{% assign currentyear = post.date | date: '%Y' %}
				{% if currentyear != year %}
					{% unless forloop.first %}
					<hr/>
					{% endunless %}
					<h3>
					<span class="fa fa-calendar" aria-hidden="true"></span>
					<a href="{{ archive_url }}#{{ currentyear }}">{{ currentyear }}</a>
					</h3>
					{% assign year = currentyear %}
				{% endif %}
				<a href="{{ archive_url }}#{{ currentdate }}"  class="btn btn-default" style="padding: 0px 5px;"><span class="fa fa-folder-open" aria-hidden="true" style="color: #1C1C1C;">{{ post.date | date: '%B' }} <!-- style="color: #1C1C1C;" is font color of span -->
				{% assign date = currentdate %}
			{% else %}
				{% assign count = count | plus: 1 %}
			{% endif %}
			{% if forloop.last %}
				<i class="badge">{{ count }}</i></span></a><hr/>
			{% endif %}
	{% endfor %}

{% for post in site.posts %}
    {% capture month %}
      {{ post.date | date: '%m%Y' }}
    {% endcapture %}
    {% capture nmonth %}
      {{ post.next.date | date: '%m%Y' }}
    {% endcapture %}
      {% capture monthHead %}
        {% if month != nmonth %}
          {% if  forloop.index != 1  %}
              {{ closeList }}
              <small markdown="1" style="padding-bottom: 35px"><!--[back to top](#top)-->
                <a href="#top" class="btn btn-default" style="font-size: 15px; padding: 0px 5px; margin-left: 30px">
                  <span class="fa fa-refresh" aria-hidden="true"></span> Go back to the top
                </a>
              </small>
              <hr/>
          {%endif %}
        <h2 class="">
            {% if year != nyear %}
          <div id="{{ post.date | date: '%Y' }}" style="padding-top:50px"></div>
          {% endif %} 
          <div id="{{ post.date | date:  '%B %Y'  }}" style="padding-top:57px"></div>
         {{ post.date | date: '%B %Y' }}
       </h2>{{ openList }}
      {% endif %}
    {% endcapture %}
    {% capture link %}
        <li>
            <a class="post-subtitle" href="{{ site.baseurl }}{{ post.url }}">
              <strong>{{ post.title }}</strong>
              <small class="post-meta"> - Posted on {{ post.date | date: "%B %-d, %Y" }}</small>
            </a>
        </li>
    {% endcapture %}
    {{ monthHead }}{{ link }}       
{% endfor %}
{{closeList}}
    <small markdown="1"><!--[back to top](#top)-->
       <a href="#top" class="btn btn-default" style="font-size: 15px; padding: 0px 5px; margin-left: 30px">
         <span class="fa fa-refresh" aria-hidden="true"></span> Go back to the top
       </a>
    </small>
    <hr/>
</div>

