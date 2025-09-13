---
layout: post
title: "Visual Scripting"
perma-link: "/havtorn/visual_scripting/"
featured-image: /assets/images/visual_scripting.gif
tags: [havtorn, editor, scripting]
date-string: July 20, 2025
---

It took a while but we have some fundamentals for a visual scripting system!
<!--excerpt-begin-->
In the example we see a physics trigger getting a ScriptComponent referencing the test script asset we authored in the separate script editor. When the block suspended in the air above overlaps with the trigger volume, we change the mesh from a block to the clock mesh, and the green point light in the foreground is toggled off. This was set up to show interactions between a few separate entities, our so called data bindings (comparable to properties in UE blueprints) and a non-trivial entry point for the script (OnBeginOverlap, instead of a simple Tick for example)
<!--excerpt-end-->

*TODO: talk about code, data binding structure, how to author nodes*