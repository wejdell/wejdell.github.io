---
layout: post
title: "Skeletal Animation"
perma-link: "/havtorn/skeletal_animation/"
featured-image: /assets/images/skeletal_animation.gif
tags: [havtorn, animation]
---

<!--excerpt-begin-->
We purposefully serialize our skeletal mesh animations down to our own data formats and run the math using those. Other solutions instead
make a thin wrapper (or no wrapping at all) around imported assimp scenes, provided they are also working with assimp. I prefer our solution,
as it seems straight forward to continue implementing more advanced features such as inverse kinematics now that we've split up the logic into
clear chunks.
<!--excerpt-end-->

*TODO: add some detail to implementation here*