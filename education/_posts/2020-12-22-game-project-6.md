---
layout: post
title: Game Project 6 - Spite Bloodloss
featured-image: /assets/images/project-6.gif
tags: [project, gameplay, vfx, rendering, ui, audio]
date-string: APRIL 06, 2021
---

<center>
	<iframe width="720" height="405" src="https://www.youtube.com/embed/5VBBRPEn-Tw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

<center>
  <b><a href="https://drive.google.com/file/d/12V6g5zM4MbOF1qL2JL7-1es_llaWa8Kw/view?usp=sharing" download="Bloodloss_Installer.exe">Download Link</a></b>
</center>

## Specifications

* **Genre**:    3D Action RPG
* **Duration**: 14 Weeks Part-Time, 4h/day
* **Engine**:   IronWrought Engine
* **Team**:     SoftBlob, 13 people

## My Contributions
<!--excerpt-begin-->
* **Gameplay** (<a href="#ability">Ability Architecture</a>, <a href="#navmesh">Navigation Mesh</a>)
* **Graphics** (<a href="#rendering">Render Pipeline</a>, <a href="#spritetext">Sprite and Text Renderers</a>, <a href="#vfx">VFX Architecture</a>, <a href="#outline">Outline Depth Stencil Pass</a>)
* **UI**	   (<a href="#ui">UI Architecture</a>, <a href="#animated">Animated Resource Orbs</a>, <a href="#dialogue">Dialogue System</a>, <a href="#floating">Floating Damage Numbers</a>, <a href="#popup">Popup Text Service</a>)
* **Audio**    (<a href="#fmod">FMod Wrapper and Audio Manager</a>)
* **Voice Over, Music**
<!--excerpt-end-->

## Details
#### <a id="ability">Ability Architecture</a>
For this project I designed the architecture surrounding abilities in the game, for the player abilities but also enemy attacks and boss abilities. Every ability was itself a game object, connected to
the player, enemy or boss game object via a separate `AbilityComponent`. Each ability had its own VFX, collider and so called behavior, defining its movement pattern if relevant. 
Each ability had its own instance pool, meaning it had a pool of instantiable objects, mostly relevant for abilities which could have multiple instances of itself active at any given time.
Abilities were activated and updated by the `AbilityComponent`, keeping track of its "parent" game object to make behaviors like a boomerang movement pattern possible. 

#### <a id="navmesh">Navigation Mesh</a>
The navigation mesh used by characters was loaded as an obj-file exported from Unity. We used Unity to generate the mesh which left a lot to be desired in terms of accuracy, but was accepted due to time constraints.
The character controllers used funneling to smoothen their paths, and utilized their own methods for choosing their destination. The player destination was determined by raycasting on the mesh and the enemies used 
the current player position for their seek behaviors or were given local randomized positions when executing their wander behaviors.

#### <a id="rendering">Render Pipeline</a>
This game was developed concurrently with our custom engine, IronWrought. I implemented the basis for our forward rendering pipeline based on our assignments in graphical programming, and extended this to encompass all the system necessary for the game.

#### <a id="spritetext">Sprite and Text Renderers</a>
Screen space quads were used for the sprite based UI, generated from points in screen space by a geometry shader. Glyphs were generated with the help of <a href="https://github.com/microsoft/DirectXTK">DirectXTKs</a> spritefont utility.
Sprites were scaled based on resolution to conserve their area on screen.

#### <a id="vfx">VFX Architecture</a>
The VFX consisted of two main elements, "VFX meshes" and particle emitters. VFX meshes were defined as models using shaders to scroll several textures over their geometry. The particle emitters used sprites for particles,
generating a screen space quad in their geometry shader. Both meshes and particle emitters were rendered with alpha blending in a late render pass. 

#### <a id="outline">Outline Depth Stencil Pass</a>
I implemented a simple depth stencil state using stenciling, and scaled the transforms of the characters to create an outline effect for the player and the current enemy hovered over by the mouse cursor. 
This also revealed the player model when it would get concealed by the walls of the environment. Simply scaling the transforms up and down for the outline is not optimal, but was very fast to get up and running.

#### <a id="ui">UI Architecture</a>
I designed the UI architecture for this project, consisting of a `Canvas` and `Button` class as its base. There was one canvas per state in the game, and it held collections of buttons, sprites and "animated UI elements". 
The canvas ran the button logic but also subscribed to messages posted by the buttons. Each button had one or more messages associated with it, which were sent when an `OnClickUp()` method was called. The buttons themselves
each had an array of sprites, each sprite associated with `Idle`, `Hover` and `Click` states. The buttons could calculate their own bounding box based on the sprite data they were initialized with, but this could also be overwritten.
The canvases were loaded from JSON documents, and the player HUD was specialized to listen to gameplay messages such as when an ability went on cooldown for example. 

#### <a id="animated">Animated Resource Orbs</a>
Inspired by <a href="https://www.youtube.com/watch?v=YPy2hytwDLM">this</a> Diablo 3 talk at GDC, I wrote shaders that would multiply and scroll different textures across the surface of a sprite.
The shader handled one to four scrolling textures for the body, one to two alpha masks which would mask the area shown, and a static overlay sprite. I used this for the health and resource globes
on the player HUD. 

A fog texture is used three times with differently scaled UV to get the roiling effect of the liquid, and a fourth is used to colorize. One mask is used as the sphere shape, and another tiling mask 
is scrolled horizontally to create the wave effect. I highlighted the glow on the very surface of the liquid by interpolating between the existing color and a defined glow color based on the UV distance to the "level" of the surface.

<center>
    <div class="photoset-grid-custom">
       <img src="/assets/images/project_6_HUD.gif">
    </div>
</center>

This same principle is used on the health and experience bars, as well as on the cooldown effects on the ability icons.

#### <a id="dialogue">Dialogue System</a>
I extended the dialogue system used in project 5 with features such as speaker portraits and titles. I also specialized it for use as the intro sequence of the game, scrolling voice line text 
across the whole screen. 

<center>
    <div class="photoset-grid-custom">
       <img src="/assets/images/project_6_dialogue.gif">
    </div>
</center>

#### <a id="floating">Floating Damage Numbers</a>
I developed a system used for spawning floating numbers above the player, for the genuine action RPG experience. Numbers were pooled and had different colors for varying strengths of
critical hits and healing. I used an analytical function defined in code to animate the size of some of the numbers to make them visually pop. I also applied a force towards the bottom of
the screen and had them randomize their starting directions, going in an arc over the center of the screen. 

<center>
    <div class="photoset-grid-custom">
       <img src="/assets/images/project_6_floating_numbers.gif">
    </div>
</center>


#### <a id="popup">Popup Text Service</a>
The floating damage numbers were part of a greater utility, called the `PopupTextService`. Apart from the damage numbers, it could also spawn fading description cards for when the player 
unlocked new skills, spawn warning text when the player's resource ran out and give instructions such as *Press Space to Continue* during dialogue breaks.

<center>
    <div class="photoset-grid-custom">
       <img src="/assets/images/project_6_popups.gif">
    </div>
</center>

Each of these instances had their own animation data, and all of these parameters were set in JSON documents.

#### <a id="fmod">FMod Wrapper and Audio Manager</a>
Since we built the engine from scratch, we needed to implement an audio interface. We decided on FMod and I wrote the basic interface towards FMod as well as the Audio Manager using it.
The logic for playing sounds was like in the project before based on messages sent by the gameplay logic to which the audio manager subscribed. All sound resources were held by the manager 
and requested from the FMod wrapper at load time. I set up FMod channels which the manager could use to group sounds and set levels for, as well as duck channels less important than others.