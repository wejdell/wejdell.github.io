---
layout: post
title: Game Project 5 - Ellah
featured-image: /assets/images/project-5.gif
tags: [project, gameplay, particle system, ui, audio]
date-string: APRIL 06, 2021
---

<center>
	<iframe width="720" height="405" src="https://www.youtube.com/embed/URmdP2C9fW0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

<center>
  <b><a href="https://drive.google.com/file/d/1DOj4Rnva5974HHe4X_qE7rS9Jccwu0Pu/view?usp=sharing" download="Ellah_Installer.exe">Download Link</a></b>
</center>

## Specifications

* **Genre**:    2D Top-Down Puzzle Adventure
* **Duration**: 8 Weeks Part-Time, 4h/day
* **Engine**:   TGA2D (developed by The Game Assembly)
* **Team**:     Diamond Eyes Studios, 12 people

## My Contributions

<!--excerpt-begin-->
* **Gameplay** (<a href="#blocks">Movable Blocks</a>, <a href="#camera">Camera System</a>, <a href="#weapons">Ice and Fire Weapon Logic</a>)
* **Graphics** (<a href="#particles">Particle System Architecture</a>, <a href="#point_lights">Point Light Logic</a>)
* **UI**       (<a href="#dialogue">Dialogue System used for Hint Signs, NPC Dialogue and Inventory Lore Text</a>)
* **Audio**    (<a href="#audio">Audio Manager and Logic</a>)
* **Voice Over, Music**
<!--excerpt-end-->

## Details
#### <a id="blocks">Movable Blocks</a>
The main mechanic of this game was to move blocks around, pushing them onto switches that would open up doors and gates. I developed much of the gameplay logic surrounding 
this feature, creating the components for basic blocks made of rock as well as gliding ice blocks that did not stop moving until they hit something. The grid based system and all
the blocking terrain and switches featured made this a bit tricky, but I think it worked out really well in the end. 

#### <a id="camera">Camera System</a>
I also developed the camera system, where a bounding box was defined for each room the player character went through. The camera view was constrained within this bounding region,
allowing for smooth transitions between the edges of a room where the player character moved around freely and the open spaces of a room where the camera was centered on and following the character.
When the player crossed one such bounding box, we would call it a *boundary transition*, entering a new room and setting a new constraining bounding box for the camera view. 
We also had *level transitions*, where the camera would fade out and a different level would be loaded.

<center>
    <div class="photoset-grid-custom">
       <img src="/assets/images/project_5_camera.gif">
    </div>
</center>

#### <a id="weapons">Ice and Fire Weapon Logic</a>
Apart from moving the blocks, the player could also change blocks. Using an ice ability, rock blocks could be frozen, transforming them into everything an icy block was. An icy 
block could also be destroyed by using a fire ability, and the fire ability could also be used light braziers on fire. On the flip side, braziers could also be extinguished using
the ice ability. This was done to enable puzzles to be designed where sometimes you needed to extinguish a flame while in others you had to ignite it. I had a major part in designing
the components needed for this behaviour.

#### <a id="particles">Particle System Architecture</a>
I designed and implemented the particle system architecture for this game, where all particles were pooled and reinitializable in runtime, enabling the fire and ice effects coming from the player
character to change directions depending on the character's facing. I also designed some of the particle systems used in the final game. Some examples include a sandstorm, a fiery crackling
from the braziers as well as an icy glinting coming off of the ice blocks. The particle system was also used in the inventory UI to mark the currently selected item. We also used it
for a dust trail coming off of the player, a sparkling coming off of item chests, and dust arising from sliding regular blocks.

<center>
    <div class="photoset-grid-custom">
       <img src="/assets/images/project_5_particles.gif">
    </div>
</center>

#### <a id="point_lights">Point Light Logic</a>
I set up components for point light lighting used for various effects, such as a fiery glow coming from the braziers, and icy sheen on the ice blocks and the point light centered on the
player character. It was important that these lights be togglable, as they had to be disabled when the fires were extinguished, or ice blocks destroyed. 


#### <a id="dialogue">Dialogue System used for Hint Signs, NPC Dialogue and Inventory Lore Text</a>
I developed a dialogue system which saw a lot of use in the game. Using the text rendering already in the engine, I set up a system that subscribed to messages sent by various components,
such as hint signs, the NPC character as well as the inventory system. The `Update()` method of this system animated the text, using a buffer to count out
the letters to the screen while keeping the formatting, as well as automatically use line breaks so that they did not have to be added to the text itself. The scrolling text could be sped
up and down by holding and releasing the spacebar, as well as skipped entirely.

<center>
    <div class="photoset-grid-custom" data-layout="3">
       <img src="/assets/images/project_5_hint.png">
       <img src="/assets/images/project_5_dialogue.gif">
       <img src="/assets/images/project_5_inventory.gif">
    </div>
</center>

#### <a id="audio">Audio Manager and Logic</a>
Finally I implemented an audio manager featuring self-made channel logic, as the TGA2D engine used Bass as its audio API. The channels could be used for grouping sound volumes, as well as
ducking channels of lesser priority to those of a higher priority. The audio logic was based on messages to which the manager subscribed, sent from various components of the gameplay system.