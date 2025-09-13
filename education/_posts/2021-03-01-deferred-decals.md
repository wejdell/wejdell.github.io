---
layout: post
title: Deferred Decals
featured-image: /assets/images/decal.gif
tags: [deferred rendering, decals, rendering]
date-string: MARCH 01, 2021
---

## Introduction

<!--excerpt-begin-->
I implemented deferred decals for our seventh game project at The Game Assembly.
The decals were exported from Unity and rendered to the G-Buffer in three separate passes depending on which textures to use; albedo, normal or "material" (containing metalness, roughness, emissive and detail normal strength).
<!--excerpt-end-->

Using three separate passes, there is no need to copy the G-Buffer and you have higher flexibility regarding which textures to use and how to alpha blend the results.
I chose to use a cube volume for projection due to its simplicity, held by the renderer and scaled by the game object transform component.

## Implementation

A decal factory receives data from the Unity export, including the filepath to a directory of textures to be used by the decal, and three flags representing which render passes to run for this specific decal. 
The albedo, material and normal (also containing ambient occlusion information) textures are requested from a material handler utility, being set to `nullptr` should they not exist rather than receiving a pointer to an error texture. 
Should any of the textures be `nullptr`, their respective flags are set to false.

The decal render pass is run right after rendering the G-Buffer. If the three decal render passes were not separated, we would set the whole G-Buffer as our render target.
If we then want to render different sets of textures for every decal, we would need to copy the whole G-Buffer and sample it in the shader so as to not overwrite the normals when we aren't interested in the normals of the decal, for example. 
The normal texture is still bound as a render target, and if we do not discard the run of the entire fragment shader (which we might not want) we are going to overwrite the G-Buffer normals.

In the below GIFs there is no copying of the G-Buffer. Decal albedo data is being written in both cases, but there is no decal data being written to the material target (left) or normals target (right). 
These are still bound as render targets though, meaning they are overwritten by default values since there is no useful data to write.

<div class="img-row">
  <div class="img-column">
    <img src="/assets/images/missing_material.gif" style="width:100%">
  </div>
  <div class="img-column">
    <img src="/assets/images/missing_normals.gif" style="width:100%">
  </div>
</div>

That said, if every decal we render renders information to the same targets, this is not an issue, and it becomes cheaper to make only one draw call.

Since we wanted the freedom to render decals to any subset of the G-Buffer, I decided to render decals in different passes to avoid copying the G-Buffer. With this setup, different render targets can be set for every shader and you are guaranteed to not overwrite. 

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>

```c++
void CDecalRenderer::Render(...)
{
    // ... Bind state common to all decals
    
    for (auto& gameObject : someDecalGameObjects)
	{
		CDecalComponent* decalComponent = gameObject->GetComponent<CDecalComponent>();

		CDecal* decal = decalComponent->GetMyDecal();
		const CDecal::SDecalData& decalData = decal->GetDecalData();

		myObjectBufferData.myToWorld = gameObject->myTransform->GetWorldMatrix();
		myObjectBufferData.myToObjectSpace = gameObject->myTransform->GetWorldMatrix().Invert();
		BindBuffer(myObjectBuffer, myObjectBufferData, "Object Buffer");

		myContext->VSSetConstantBuffers(1, 1, &myObjectBuffer);
		myContext->PSSetConstantBuffers(1, 1, &myObjectBuffer);
		myContext->PSSetShaderResources(5, 3, &decalData.myMaterial[0]);

                if (decalData.myShouldRenderAlbedo)
                {
                    myContext->PSSetShader(myAlbedoPixelShader, nullptr, 0);
                    myContext->DrawIndexed(myNumberOfIndices, 0, 0);
                    CRenderManager::myNumberOfDrawCallsThisFrame++;
                }
        
                if (decalData.myShouldRenderMaterial)
                {
                    myContext->PSSetShader(myMaterialPixelShader, nullptr, 0);
                    myContext->DrawIndexed(myNumberOfIndices, 0, 0);
                    CRenderManager::myNumberOfDrawCallsThisFrame++;
                }

                if (decalData.myShouldRenderNormals)
                {
                    myContext->PSSetShader(myNormalPixelShader, nullptr, 0);
                    myContext->DrawIndexed(myNumberOfIndices, 0, 0);
                    CRenderManager::myNumberOfDrawCallsThisFrame++;
                }
	}
}
```

</details>
{::options parse_block_html="false" /}

Then comes the issue of alpha blending. Setting up alpha blending between one source and one destination is simple, but as our material target of the G-Buffer (which is *not* identical to the material texture of the decal) contains metalness, roughness, emissive and ambient occlusion, one has to be sacrificed as the alpha blending channel.

In the current implementation, I chose to sacrifice ambient occlusion on the decals and to sample the alpha level from the albedo texture. This means that even though we don't render the decal to the albedo target, an albedo shader resource must still be bound. 
For the normals, the fourth component is unoccupied (since we moved the ambient occlusion from the normal texture to the material target), and also samples from the albedo shader resource.
The graphical artists could work the transparency into the material and normal textures for decals, but I found this solution better suited for our needs. 

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>

```hlsl
// Decal_MaterialPixelShader.hlsl
#include "DecalShaderStructs.hlsli"

float4 main(VertexToPixel input) : SV_TARGET3
{
    float3 clipSpace = input.myClipSpacePosition;
    clipSpace.y *= -1.0f;
    float2 screenSpaceUV = (clipSpace.xy / clipSpace.z) * 0.5f + 0.5f;
    
    float z = depthTexture.Sample(defaultSampler, screenSpaceUV).r;
    float x = screenSpaceUV.x * 2.0f - 1;
    float y = (1 - screenSpaceUV.y) * 2.0f - 1;
    float4 projectedPos = float4(x, y, z, 1.0f);
    float4 viewSpacePos = mul(toCameraFromProjection, projectedPos);
    viewSpacePos /= viewSpacePos.w;
    float4 worldPosFromDepth = mul(toWorldFromCamera, viewSpacePos);
    
    float4 objectPosition = mul(toObjectSpace, worldPosFromDepth);
    clip(0.5f - abs(objectPosition.xyz));
    float2 decalUV = objectPosition.xy + 0.5f;
    decalUV.y *= -1.0f;

    float3 material = materialTexture.Sample(defaultSampler, decalUV).rgb;
    float alpha = albedoTexture.Sample(defaultSampler, decalUV).a;
	
    return float4(material.rgb, alpha);
}
```

</details>
{::options parse_block_html="false" /}

The three screen shots below depict a decal being rendered to different parts of the G-Buffer. Left is only albedo, using the material and normal information already stored in the G-Buffer, middle is only material, and right is only normals.

<div class="img-row">
  <div class="img-column">
    <img src="/assets/images/decal_albedo.jpg" style="width:100%">
  </div>
  <div class="img-column">
    <img src="/assets/images/decal_material.jpg" style="width:100%">
  </div>
  <div class="img-column">
    <img src="/assets/images/decal_normal.jpg" style="width:100%">
  </div>
</div>