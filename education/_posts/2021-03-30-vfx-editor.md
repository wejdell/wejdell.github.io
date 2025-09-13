---
layout: post
title: "VFX Editor using ImGui"
featured-image: /assets/images/vfx_editor_gif.gif
tags: [vfx, imgui, tools]
date-string: MARCH 30, 2021
---

<script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<script>window.jQuery || document.write('<script src="_/js/libs/jquery-1.9.1.min.js"><\/script>')</script>

## Introduction

<!--excerpt-begin-->
For our seventh game project at The Game Assembly, I made a VFX editor using <a href="https://github.com/ocornut/imgui">ImGui</a> based on the VFX and particle systems I designed for the project prior.
 With ImGui it was easy and fast to get a graphical user interface running and rendering in our custom engine, and features like the color picker and curve editing were essential in the workflow designed for the graphical artists.
<!--excerpt-end-->

This editor was made to provide the artists working with VFX a proper pipeline for doing so, as tweaking values in JSON files that they had been doing up to this point was inconvenient at best.
The systems were still initialized from JSON but did not have to be tweaked inside the files themselves. 

## Implementation

The VFX Editor works on a component called `VFXSystemComponent`. This component holds a list of data structures called `VFXEffect`s. 
Each effect holds data associated with any number of what we call VFX meshes and particle emitters, where a VFX mesh is simply some geometry using shaders to scroll textures across it. These meshes and particle emitters are rendered using alpha blending and with depth writing turned off in a late render pass.
The system is defined this way to allow a single game object with this component to store several collections of meshes and particle emitters in different configurations, and to activate any one of these collections as an "effect" at any time.

The window class itself holds the data necessary to tweak values in ImGui as well as the data necessary to write to JSON. All data structures have their own, unique `Serialize()` methods in order to conserve the specific format used before, rather than generalizing and having to rewrite the JSON import for the `VFXSystemComponent`.

The basic logic is very simple. An enum controls which information is shown in the window, and the enum is set by buttons within the window.

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>

```c++
void IronWroughtImGui::CVFXEditorWindow::OnInspectorGUI()
{
	ImGui::Begin(Name(), Open());

	switch (myCurrentMenu)
	{
	case IronWroughtImGui::EVFXEditorMenu::MainMenu:
		ShowMainMenu();
		break;
	case IronWroughtImGui::EVFXEditorMenu::VFXMeshView:
		ShowVFXMeshWindow();
		break;
	case IronWroughtImGui::EVFXEditorMenu::ParticleEmitterView:
		ShowParticleEffectWindow();
		break;
	default:
		break;
	}

	ImGui::End();
}
```

</details>
{::options parse_block_html="false" /}

On pressing save, all data structures currently held by the window are serialized, written to a filepath set by the user, and the singular VFXSystemComponent is reinitialized using the new or modified JSON file.

<a href="/assets/images/vfx_save.gif"><img class="centered full" src="/assets/images/vfx_save.gif"></a>

In the main window, VFX systems can be loaded and modified on the highest level. Effects can be added, and for every effect you can add any amount of VFX meshes and particle emitters. These can be offset and rotated about the associated game object, and each have their own delay and duration.
Any mesh or particle emitter file can be modified by a button press, where the view changes to show the file name and all attributes associated with the selected mesh or emitter. Saving works the same way in this view, but the file name of the selected mesh or emitter can also be changed here. 

<a href="/assets/images/vfx_open_separate_files.gif"><img class="centered full" src="/assets/images/vfx_open_separate_files.gif"></a>

For the particle emitter file view, there are tweakable curves for color, size and direction of the particles. These curves are opened in a separate window. For every attribute that can be changed by a curve, there is an `A` and a `B` value, such as starting color and final color. 
The curves are defined in the interpolation parameter (`y`-axis) over time (`x`-axis) coordinate system, meaning if the `y`-value at `x = 0` is `0`, the particle starts its lifetime at attribute value `A`, and if `y = 1`, the particle starts its lifetime at attribute value `B`.

In the current version of the editor, the curves are not actually curved, but straight lines going through several points. We had trouble determining how to interpret smooth line segments on the engine side when we first started setup, 
and so resorted to interpolating between easily definable points instead due to the time constraint. 

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>

```c++
const float SVFXEffect::CalculateInterpolator(const std::vector<Vector2>& somePoints, const float t) const
{
	unsigned int pointIndex = static_cast<unsigned int>(somePoints.size() - 1);
	
	for (unsigned int j = 1; j < somePoints.size() - 1; ++j)
	{
		if (t < somePoints[j].x && t > somePoints[j - 1].x)
		{
			pointIndex = j;
			break;
		}
	}
	
	// Nice function inspired by Freya Holmer, inverse lerps from one range and lerps into another
	float val = Remap(somePoints[pointIndex - 1].x, somePoints[pointIndex].x, 0.0f, 1.0f, t);
	return Lerp(somePoints[pointIndex - 1].y, somePoints[pointIndex].y, val);
}

void SVFXEffect::UpdateParticles(unsigned int anIndex, CParticleEmitter::SParticleData& particleData)
{
	// ...

	for (UINT i = 0; i < myParticleVertices[anIndex].size(); ++i)
	{
		float quotient = myParticleVertices[anIndex][i].myLifeTime / particleData.myParticleLifetime;		
	
		if (!particleData.myColorCurve.empty())
		{
			myParticleVertices[anIndex][i].myColor = Vector4::Lerp
			(
				particleData.myParticleStartColor,
				particleData.myParticleEndColor,
				CalculateInterpolator(particleData.myColorCurve, quotient)
			);
		}

		// ... Continue updating particles
	}
}
```

</details>
{::options parse_block_html="false" /}

When a curve is saved, the system is reinitialized.

<a href="/assets/images/vfx_size_curve.gif"><img class="centered full" src="/assets/images/vfx_size_curve.gif"></a>

## Improvements

Considering the editor was made under time constraints, there are a lot of improvements that could be made, some of which are listed below.

* Velocity and opacity curves for particle emitter
* Scroll speed, opacity and size curves for vfx meshes
* Full PBR support for textures scrolled over vfx mesh, and particle textures
* Randomizable rotation for particle textures and meshes
* Easier removal and adding of systems, effects, vfx meshes and particle emitters

Hopefully there will be time to realize these during our final exam project.

<script src="/assets/js/jquery.photoset-grid.js"></script>
<script type="text/javascript">
    $('.photoset-grid-custom').photosetGrid({
    // Set the gutter between columns and rows
    gutter: '5px',
  
    // Wrap the images in links
    highresLinks: true,
  
    // Asign a common rel attribute
    rel: 'print-gallery',

    onInit: function(){},
    
    onComplete: function(){
        // Show the grid after it renders
        $('.photoset-grid-custom').attr('style', '');
    }
});
</script>