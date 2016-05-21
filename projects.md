---
layout: page
title: Projects
permalink: /projects/
banner_image: sample-banner-image-3.jpg
---

<div>
  {% for project in site.projects %}
    {% capture currentyear %}{{project.date | date: "%Y"}}{% endcapture %}
    {% if currentyear != year %}
      {% unless forloop.first %}
      </ul>
      {% endunless %}
      <h5>{{ currentyear }}</h5>
      <ul>
      {% capture year %}{{currentyear}}{% endcapture %} 
    {% endif %}
    <li><a href="{{ project.url | prepend: site.baseurl }}">{{ project.title }}</a></li>
{% endfor %}
</div>

{% for project in site.tags.Play %}
<h2>{{ project.title }}</h2>
<time>{{ project.date }}</time>
{% endfor %}