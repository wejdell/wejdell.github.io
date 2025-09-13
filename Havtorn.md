---
title: "Project Havtorn"
permalink: "/havtorn/"
layout: default
---

<article>
    {% include meta.html post=page %}
    <md-block>


        Havtorn is a <a href="https://github.com/wejdell/Havtorn">game engine</a> with a custom, integrated editor that I started building in my spare time,
        starting in 2021. The goal was always to develop games with it, though the greatest rewards so far have been the cultivation of skills in building a thought-out,
        usable and modular game engine architecture.

        Despite most of the work being done by myself exclusively - in late evenings after a full day's work - <a href="https://www.linkedin.com/in/axel-savage-bb48b598/">Axel Savage</a>
        and <a href="https://www.linkedin.com/in/gonzalez-aki/">Aki Gonzalez</a> have always been my biggest supporters, and have done work on the project when able. We now have a 
        small group of former class mates from <a href="/education/">The Game Assembly</a> working on the first project.

        Please have a look at some of the highlights of the development below, or find the repo <a href="https://github.com/wejdell/Havtorn">here</a>.


    </md-block>
    <script type="module" src="https://md-block.verou.me/md-block.js"></script>
</article>

  {% for post in site.categories.havtorn %}
<article>
    {% include meta.html post=post %}
    {% assign excerptParts = post.excerpt | split: "<!--excerpt-begin-->" %}
    {{ excerptParts[1] | strip_newlines }}
    <footer class="button"><a href="{{ post.url | relative_url }}">read more</a></footer>
</article>
  {% endfor %}