---
layout: post
title: Game Project 7 - I Am
perma-link: "/education/project-7/"
featured-image: /assets/images/project-7.gif
tags: [project, deferred rendering, rendering, ui, vfx, audio]
date-string: APRIL 06, 2021
---

## Specifications

* **Genre**:    3D FPX
* **Duration**: 16 Weeks Part-Time, 4h/day
* **Engine**:   IronWrought Engine
* **Team**:     SoftBlob, 16 people

## My Contributions
<!--excerpt-begin-->
* **Graphics** (<a href="#hdr">HDR</a>, <a href="#volumetric_lighting">Volumetric Lighting</a>, <a href="#material">Material Pipeline</a>, <a href="#deferred_decals">Deferred Decals</a>, <a href="#vfx_editor">VFX Editor</a>, <a href="#vertex_paint">Vertex Painting</a>)
* **UI**       (<a href="#animated_sprites">Animated sprites</a>, <a href="#ui_architecture">Extended Architecture</a>)
* **Audio**    (<a href="#voice_line_event">Voice Line Event and Category Nodes</a>)
* **Voice Over**
<!--excerpt-end-->
## Details
#### <a id="hdr">HDR</a>

Starting on project 7 we moved to a deferred rendering pipeline instead of forward rendering. I took the chance to implement HDR lighting at the same time, using the filmic tonmapping equation of <a href="https://www.gdcvault.com/play/1012351/Uncharted-2-HDR">Uncharted 2</a>.
This vastly enhanced the quality of our emissive lights and improved our control over the lighting over all.

#### <a id="volumetric_lighting">Volumetric Lighting</a>
Please see the <a href="{{site.baseurl}}/education/specialization/">post</a> I made about my specialization project at The Game Assembly.

#### <a id="material">Material Pipeline</a>
For project 7, it was important that we could have several sets of materials for any model. I worked closely together with the graphical artists to develop a system which takes care of all material texture memory, using reference counting as well as applying error textures where applicable. 
Material names where applied in Maya and subsequently detected during import of the FBX file. The Material Handler used a single material name to load the relevant textures. We made use of a material suffix to detect which models were to be rendered with alpha blending at a later pass. 
This Material Handler was also used to handle decal textures and materials used for vertex painting. 

#### <a id="deferred_decals">Deferred Decals</a>
Please see the <a href="https://nicolas-risberg.github.io/2021-03-01/deferred-decals.html">post</a> I made about deferred decals in the highlights.

#### <a id="vfx_editor">VFX Editor</a>
Please see the <a href="https://nicolas-risberg.github.io/2021-03-30/vfx-editor.html">post</a> I made about the VFX editor in the highlights.

#### <a id="vertex_paint">Vertex Painting</a>
I worked with <a href="http://axelsavage.com">Axel Savage</a> on implementing vertex painting from Unity. On the Unity side we used <a href="https://unity3d.com/unity/features/worldbuilding/polybrush">Polybrush</a> to paint data on the meshes, and the per-vertex data was
exported to our engine. The order of indices in the index buffer turned out to not be the same when the data came from Unity, so we resorted to making a hash function for the vertex positions in order to correctly map position to color using the order we loaded from the FBX file.

In the pixel shader, all values associated with the PBR materials were interpolated using the color values as weights. 

<img class="centered full" src="/assets/images/vertexPaint.gif">

#### <a id="animated_sprites">Animated Sprites</a>
I extended the sprite rendering I developed in project 6 to support spritesheet animations, breathing life into the crosshair especially. Animations were relatively easy to define in the json document already containing the sprite information, and the logic was general enough to make
simple function calls like `mySprite->PlayAnimation(anAnimationIndex, doRepeat, doReverse);` behave as expected. Animations were given support for transitioning to different animations depending on whether they played in reverse or not. They could also be given a rotational speed
about the $$z$$-axis, as seen in <a href="/assets/images/project-7.gif">this gif</a>. 

The animation logic consisted of counting up the index of a collection of all relevant UV-coordinates, all laid out in sequence. What defines an animation is basically its offset from the beginning of the collection and its total number of frames.

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>

```json
...
"Animations": [
{
    "Name": "Shoot",
    "FrameWidth": 240.0,
    "FrameHeight": 240.0,
    "VerticalStartingPos": 0.0,
    "FrameOffset": 0,
    "NumberOfFrames": 7,
    "FramesPerSecond": 60,
    "RotationSpeedPerSecond": 720.0,
    "ShouldLoop": false,
    "TransitionIndex": 2,
    "ReverseTransitionIndex": 1
},
...
]
```

```c++
void CSpriteInstance::Update()
{
	if (!myShouldAnimate)
		return;

	if (!myShouldReverseAnimation)
		this->Rotate(myAnimationData[myCurrentAnimationIndex].myRotationSpeedInSeconds * CTimer::Dt());
	else 
		this->Rotate(-myAnimationData[myCurrentAnimationIndex].myRotationSpeedInSeconds * CTimer::Dt());

	if ((myAnimationTimer += CTimer::Dt()) > (1.0f / myAnimationData[myCurrentAnimationIndex].myFramesPerSecond))
	{
		myAnimationTimer -= (1.0f / myAnimationData[myCurrentAnimationIndex].myFramesPerSecond); 

		if (!myShouldReverseAnimation)
		{
			myCurrentAnimationFrame++;
			if (myCurrentAnimationFrame > (myAnimationData[myCurrentAnimationIndex].myNumberOfFrames + myAnimationData[myCurrentAnimationIndex].myFramesOffset - 1))
			{
				myShouldAnimate = myShouldLoopAnimation;
				if (!myShouldAnimate)
				{
					PlayAnimationUsingInternalData(myAnimationData[myCurrentAnimationIndex].myTransitionToIndex);
					return;
				}

				myCurrentAnimationFrame = myAnimationData[myCurrentAnimationIndex].myFramesOffset;
			}
		}
		else 
		{
			// ...
		}

		this->SetUVRect(myAnimationFrames[myCurrentAnimationFrame]);
	}
}
```

</details>
{::options parse_block_html="false" /}

#### <a id="ui_architecture">Extended UI Architecture</a>
The UI architecture was extended to use "widgets" in the main menu. Rather than a state change on a button press, a child canvas of the main canvas is activated, each potentially holding its own buttons and sprites. 
We added a canvas wide offset to such children to properly set the render order of image elements. We limited ourselves to let the button logic only really apply to the base canvas though, so button presses in the widgets
were only really relevant to the base canvas. Since we only used this system in the main menu we were not negatively affected by this limitation, but to really make use of widgets we would have to improve it.

#### <a id="voice_line_event">Voice Line Event and SFX Category Nodes</a>
Some of the other team members, especially <a href="http://haqvinbager.github.io">Haqvin Bager</a> built a node editor using <a href="https://github.com/ocornut/imgui">ImGui</a> for this project.
I contributed to this editor by making nodes that would play sounds, for playing voice lines on event triggers in the levels as well as playing randomized SFX and reactionary voice lines based on certain categories. 
Categories included `RobotDeath`, `PlayerStepAirVent` and `ResearcherPickUpExplosivesReaction` among others.
The audio manager would load lists of clips based on each category on start up, and randomize (cycling through the whole list before replaying) what clip to play when receiving a message from the node editor.
