---
layout: post
title: Game Project 8 - Besoket
perma-link: "/education/project-8/"
featured-image: /assets/images/project-8.gif
tags: [project, rendering, ui, audio]
date-string: JUNE 18, 2021
---

<center>
	<iframe src="https://www.youtube.com/embed/R9eKjTaD22Y?si=tvvROig8QxMoyu7q" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</center>

<center>
<div class="iframe itchIframe">
	<iframe src="https://itch.io/embed/1109245" frameborder="0"><a href="https://the-game-assembly.itch.io/tga19-p8-besoket">TGA19 Game Project 8 - Besoket by The Game Assembly</a></iframe>
</div>
</center>

<br>

<!--excerpt-begin-->
<center>
<i>You respond to a cry of help from your friend, locked in a strange Swedish cottage. Find a way to free your friend, solve the puzzles all while evading the unknown horrors of the house. You better be careful, or this act of kindness might be your last..</i>
</center>
<!--excerpt-end-->

<br>

Our exam project turned out fairly popular, with thousands of views on <a href="https://the-game-assembly.itch.io/tga19-p8-besoket">itch.io</a> and more than 2000 downloads! Please enjoy the jank if you feel like trying it.

<br>

## Specifications

* **Genre**:    Horror FPX
* **Duration**: 9 Weeks Full-Time, 8h/day
* **Engine**:   IronWrought Engine
* **Team**:     SoftBlob, 16 people

## My Contributions
* **Graphics** (<a href="#spotlights">Spot Lights</a>, <a href="#camera">Camera Improvements</a>)
* **UI**       (<a href="#vignette">UI Improvements</a>)
* **Audio**    (<a href="#spatialAudio">3D Audio and Cone Based Occlusion</a>)
* **Voice Over & Sound Design**

## Details
#### <a id="spotlights">Spot Lights</a>

While missing shadowmapping for them, I added support for spotlights imported from Unity (which we used as a level editor during the final school projects) into the engine. They really contributed to the ambience
of the basement levels.

<div class="img-row">
  <div class="img-column">
      <div class="click-zoom">
        <label>
            <input type="checkbox" />
            <img class="centered" src="/assets/images/project_8-spotlight-import.gif">
        </label>
      </div>
  </div>
  <div class="img-column">
      <div class="click-zoom">
        <label>
            <input type="checkbox" />
            <img class="centered" src="/assets/images/besoket-1.png">
        </label>
      </div>
  </div>
</div>

#### <a id="camera">Camera Improvements</a>

Our reference game this time was Soma, and pulling from its' ancestor Amnesia, we knew we wanted to get a little bit of sway into
the camera movement. We didn't use very violent shakes in the end but the gentle sway simulating the character breathing is felt
ambiently and helps immersing the player.

<div class="img-row">
  <div class="img-column">
      <div class="click-zoom">
        <label>
            <input type="checkbox" />
            <img class="centered" src="/assets/images/project_8-subtle-bob.gif">
        </label>
      </div>
  </div>
  <div class="img-column">
      <div class="click-zoom">
        <label>
            <input type="checkbox" />
            <img class="centered" src="/assets/images/project_8-crazy-shake.gif">
        </label>
      </div>
  </div>
</div>

We were also aiming to play more with the main menu than we had before. As part of that, we framed a few different assets nicely
and let them represent different submenus. Below is an in-progress shot of the camera moving between a few such points during 
development.

<div class="click-zoom">
<label>
    <input type="checkbox" />
    <img class="centered" src="/assets/images/project_8-menu-interpolation.gif">
</label>
</div>

#### <a id="vignette">UI Improvements</a>

Also as part of invoking the visual effects of Soma and Amnesia, I built some controls for the artists to tweak the vignette 
overlay which was later animated when killed by the monster in the game.

<div class="img-row">
  <div class="img-column">
      <div class="click-zoom">
        <label>
            <input type="checkbox" />
            <img class="centered" src="/assets/images/project_8-post-processing.gif">
        </label>
      </div>
  </div>
  <div class="img-column">
      <div class="click-zoom">
        <label>
            <input type="checkbox" />
            <img class="centered" src="/assets/images/project-8.gif">
        </label>
      </div>
  </div>
</div>


#### <a id="spatialAudio">3D Audio and Cone Based Occlusion</a>

I extended the audio sources with basic spatialization in the final project. For occlusion, I introduced separate angles to 
the prefab in Unity so we could simulate basic occlusion through walls. The attenuation distances (unattenuated to fully attenuated)
and cone angles were then provided to FMOD on initialization.



