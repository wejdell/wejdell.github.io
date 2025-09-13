---
title: "The Game Assembly"
permalink: "/education/"
layout: default
---

<article>
  
  <img class="centered half" src="/assets/images/tga.png">
  {% include meta.html post=page %}
  <md-block>
  

  I attended The Game Assembly in Stockholm between 2019 and 2021 enrolled in the <a href="https://thegameassembly.com/se/utbildningar/spelprogrammerare/">game programmer</a> program. 
  This page serves as an archive of what I produced and achieved in the last half of the program (including <a href="{{site.baseurl}}/education/specialization/">specialization</a>)
  before starting my internship at <a href="{{site.baseurl}}/tarsier/">Tarsier</a>.
  

  </md-block>
  <script type="module" src="https://md-block.verou.me/md-block.js"></script>
</article>

  {% for post in site.categories.education %}
<article>
    {% include meta.html post=post %}
    {% assign excerptParts = post.excerpt | split: "<!--excerpt-begin-->" %}
    {{ excerptParts[1] | strip_newlines }}
    <footer class="button"><a href="{{ post.url | relative_url }}">read more</a></footer>
</article>
  {% endfor %}