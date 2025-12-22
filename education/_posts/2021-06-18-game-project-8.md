---
layout: post
title: Game Project 8 - Besoket
perma-link: "/education/project-8/"
featured-image: /assets/images/project-8.gif
tags: [project, rendering, ui, audio]
date-string: JUNE 18, 2021
---

<center>
	<iframe width="720" height="405" src="https://www.youtube.com/embed/R9eKjTaD22Y?si=tvvROig8QxMoyu7q" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</center>

<center>
	<iframe src="https://itch.io/embed/1109245" width="720" height="167" frameborder="0"><a href="https://the-game-assembly.itch.io/tga19-p8-besoket">TGA19 Game Project 8 - Besoket by The Game Assembly</a></iframe>
</center>

<br>
Our exam project turned out fairly popular, with thousands of views on <a href="https://the-game-assembly.itch.io/tga19-p8-besoket">itch.io</a> and more than 2000 downloads! Please enjoy the jank if you feel like trying it.
<br>


## Specifications

* **Genre**:    Horror FPX
* **Duration**: 9 Weeks Full-Time, 8h/day
* **Engine**:   IronWrought Engine
* **Team**:     SoftBlob, 16 people

## My Contributions
<!--excerpt-begin-->
* **Graphics** (<a href="#spotlights">Spot Lights</a>, <a href="#camera">Camera Improvements</a>)
* **UI**       (<a href="#vignette">UI Improvements</a>)
* **Audio**    (<a href="#spatialAudio">3D Audio</a>, <a href="#occlusion">Cone Based Occlusion</a>)
* **Voice Over**
<!--excerpt-end-->

## Details
#### <a id="hdr">HDR</a>

Starting on project 7 we moved to a deferred rendering pipeline instead of forward rendering. I took the chance to implement HDR lighting at the same time, using the filmic tonmapping equation of <a href="https://www.gdcvault.com/play/1012351/Uncharted-2-HDR">Uncharted 2</a>.
This vastly enhanced the quality of our emissive lights and improved our control over the lighting over all.

