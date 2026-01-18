---
layout: post
title: "Render Based Picking"
perma-link: "/havtorn/render_based_picking/"
featured-image: /assets/images/renderBasedPicking.gif
tags: [havtorn, editor]
---

*Picking*, in a game engine architecture context, is the term we use to describe the process of clicking on actors or entities in a scene to select, or manipulate them somehow. 
This is often done by shooting a trace or raycast into the scene from the mouse cursor when clicking, using the physics layer, and selecting the entity corresponding to whatever collision body that the ray intersects with, depending on the collision rules set up. 
Rather than using this approach, 
<!--excerpt-begin-->
Havtorn uses information stored in a separate render target to resolve its picking in the editor. By rendering the identifier of each entity onto the screen, our picking becomes pixel perfect and completely decoupled from the physics system. 
<!--excerpt-end-->

One nice feature of an ECS architecture is the fact that you can represent an entity and reference all of its data using only an ID. There's no need to get hold of a pointer to a large actor object, you can simply provide a single data type directly. 
We utilize this fact by extending the <a href="https://en.wikipedia.org/wiki/Deferred_shading">G-Buffer</a> with a separate target to store the ID in, per pixel. 64 bits per ID is enough for us so we make it a 64 bit texture, using the `DXGI_FORMAT_R32G32_UINT` format of DirectX. 
When rendering to the G-Buffer, we then just pass along the entity ID of every instance that gets rendered. To solve picking for transparent and invisible objects, I opted to represent them as world space billboard sprites. They are also drawn to the G-Buffer, only after the lighting pass. 

<div class="click-zoom">
    <label>
        <input type="checkbox" />
        <img class="centered" src="/assets/images/worldPositionSpriteWidgets.png">
    </label>
</div>

My thinking is that we can hopefully find a way to represent any entity in the game world like this, either with solid geometry directly or by using one of these sprites. There are definitely challenges in how to scale and layer them so they stay visible, and there are probably also specific cases where this 
solution breaks down or fails to accurately represent some type of entity, but I think it's worth it in the end to reach the goal of clicking what you see in the viewport to select it. I always get incredibly frustrated when I accidentally click a fog cube instead of the thing I'm actually trying to select in 
the Unreal Editor because I've failed to toggle off translucent object selection.

It should be noted that this is only for use in editor time. Extending the G-Buffer is costly and we don't want to deal with any editor functionality at all in a finished game, if we can help it. Separate render passes are chosen here depending on the play state of the game.

The debug view in the gif above is rendered using a simple formula, taking the average of the red and green channel (making up the complete 64 bit entity ID) for the blue, and then scaling the whole thing by some arbitrary amount to where you can tell most of the entities apart.

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>

```c++
// FullscreenEditorData_PS.hlsl
...

Texture2D<uint2> entityDataTexture : register(t0);

PixelOutput main(VertexToPixel input)
{
    PixelOutput returnValue;
    
    const uint2 resource = entityDataTexture.Load(int3(input.UV.xy * Resolution, 0));
    returnValue.Color.rgb = float3(resource.x, resource.y, (resource.x + resource.y) * 0.5f);
    returnValue.Color.rgb /= 1000000000;
    returnValue.Color.a = 1;
    return returnValue;
};
```

</details>
{::options parse_block_html="false" /}

## Dragging Entities into the Viewport

<div class="click-zoom">
    <label>
        <input type="checkbox" />
        <img class="centered" src="/assets/images/renderBasedDragging.gif">
    </label>
</div>

A similar approach can also be used to resolve where to put a preview entity you're dragging in from the asset browser. Again, you can do this by checking collision with the physics system, but I think it's entirely serviceable 
to render out the world position to another render target, and sample from that one on the CPU in the same way as with the entity IDs.

<div class="click-zoom">
    <label>
        <input type="checkbox" />
        <img class="centered" src="/assets/images/worldPositionTexture.png">
    </label>
</div>

A key consideration here is how to deal with the geometry of the entity you're dragging, otherwise it will jump around or travel towards the camera as the cursor position hovers over it. We solve this by just not rendering the 
preview entity to the world position render target, at least until you let go of the cursor and it becomes a proper entity in the scene. Where there is no geometry or sprite rendered (such as over the skybox), you can put it at a set distance from the camera along the vector separating the camera and the cursor. 