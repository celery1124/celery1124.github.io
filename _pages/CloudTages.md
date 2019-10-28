---
layout: page
title: Blog List With Tags
permalink: /CloudTags/
---

{: #top }

<!-- this code si from https://github.com/daattali/daattali.github.io/blob/master/index.html --> 
<div class="list-filters post-preview" style="text-align:center;font-family:Helvetica;">
  <a href="/" class="list-filter"> All posts </a> &nbsp;
  <a href="/CloudCategories" class="list-filter filter-selected"> Catergories </a> &nbsp;
  <a href="/CloudTags" class="list-filter"> Tags </a> &nbsp;
  <a href="/CloudDate" class="list-filter"> By Date </a> &nbsp;
</div>

<!-- this code from https://github.com/codinfox/codinfox-lanyon/blob/dev/blog/tags.html-->
<!-- class="later on" means I will design again -->
<!-- class="later on" is changed while seeing my github page of index.html--> 

  <div class="blog-tags"> <!-- blog-tags-->
    {% assign tags = site.tags | sort %}
    {% for tag in tags %} <!--"#{{ tag[0] | slugify }}"--> 
    <a href="#{{ tag[0] | slugify }}" class="btn btn-default" style="font-size: {{ tag | last | size  |  times: 4 | plus: 80  }}%"> <!-- style="color: #1C1C1C;" is font color of cloud index -->
      <span class="fa fa-folder-open" aria-hidden="true" style="color: #1C1C1C;"> <!-- I got rid of left option-->
        {{ tag[0] }} <i class="badge">{{ tag | last | size }}</i>
      </span>
    </a>
    {% endfor %}
  </div>
  <hr/> <!-- margin-top and margin-bottom in main.css -->
  <div class="post-preview"> <!--post-preview and maring-bottom is what I need-->
    {% for tag in tags %} <!-- style="padding-top: 70px;" is used to deal with nav-bar -->
      <h2 id="{{ tag[0] | slugify }}" style="padding-top: 70px;"> {{ tag[0] }}  <i class="badge">{{ tag | last | size }}</i></h2> <!-- I added new class -->
      <ul class="later on"> <!-- post-subtitle -->
        {% for post in tag[1] %}
          <a class="post-subtitle" href="{{ site.baseurl }}{{ post.url }}"><!-- I think I have to find css of class ou, first of all, I use post-title--> <!-- I think I don't need class of a tag in here -->
        <li>
        {{ post.title }}
        <!-- <p class="post-meta"></p> in index.thml -->  
        <small class="post-meta"> - Posted on {{ post.date | date: "%B %-d, %Y" }}</small>
        </li>
        </a>
        {% endfor %}
      </ul>
        <a href="#top" class="btn btn-default" style="font-size: 15px; padding: 0px 5px; margin-left: 30px">
          <span class="fa fa-refresh" aria-hidden="true"></span> Go back to the top
        </a>  
        <hr/>
    {% endfor %}
  </div>