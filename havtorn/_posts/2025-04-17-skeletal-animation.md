---
layout: post
title: "Skeletal Animation"
perma-link: "/havtorn/skeletal_animation/"
featured-image: /assets/images/newSkelAnimControls.gif
tags: [havtorn, animation]
---

<!--excerpt-begin-->
We purposefully serialize our skeletal mesh animations down to our own data formats and run the math using those. Other solutions instead
make a thin wrapper (or no wrapping at all) around imported assimp scenes, provided they are also working with assimp. I prefer our solution
as it gives us more control in implementing more advanced features such as inverse kinematics now that we've split up the logic into clear chunks.
<!--excerpt-end-->

Our flow currently looks like this. We have a collection of thin `PlayData` objects that refer to distinct animation assets. The thought is that 
whenever we want play an animation, we just push one of these objects into the animation component's collection. We start by resolving the local 
bone pose for each of the ones currently playing by recursively traversing the node graph. After that, all the `PlayData` local poses are blended together.
Following a final traversal of the tree where we apply the blended local pose, we iterate through the posed nodes and simply multiply the inverse bind pose
to get our final world space bone transform, and send that off to the shader.

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>

```c++
// CAnimationGraphSystem.cpp
...

// Read local poses of playing animations
for (SSkeletalAnimationPlayData& playData : component->PlayData)
{
    if (!UMath::IsWithin(playData.AssetReferenceIndex, 0u, STATIC_U32(component->AssetReferences.size())))
      continue;
    
    const U32 index = component->AssetReferences[playData.AssetReferenceIndex];
    const SSkeletalAnimationAsset* animationAsset = assetRegistry->RequestAssetData<SSkeletalAnimationAsset>(index, component->Owner.GUID);
    if (animationAsset == nullptr)
      continue;
  
    ...
  
    playData.LocalPosedNodes = {};
    ReadAnimationLocalPose(animationAsset, meshAsset, animationTime, meshAsset->Nodes[0], playData.LocalPosedNodes);
}
				
std::vector<SSkeletalPosedNode> posedNodes = {};
posedNodes.resize(meshAsset->Nodes.size());
				
// Blend animations
if (component->PlayData.size() > 1)
{
    // TODO.NW: Barycentric interpolation for more than 2 animations
  
    for (U32 i = 0; i < STATIC_U32(posedNodes.size()); i++)
    {
        const SSkeletalPosedNode& posedNodeA = component->PlayData[0].LocalPosedNodes[i];
        const SSkeletalPosedNode& posedNodeB = component->PlayData[1].LocalPosedNodes[i];
    
        posedNodes[i].Name = meshAsset->Nodes[i].Name;
        posedNodes[i].LocalTransform = SMatrix::Interpolate(posedNodeA.LocalTransform, posedNodeB.LocalTransform, component->BlendValue);
    }
}
else if (component->PlayData.size() > 0)
{
    posedNodes = component->PlayData[0].LocalPosedNodes;
}

// Apply local pose and inverse bind transform
SMatrix root = SMatrix::Identity;
root.SetScale(importScale);
ApplyLocalPoseToHierarchy(meshAsset, posedNodes, meshAsset->Nodes[0], root);

component->Bones.resize(meshAsset->BindPoseBones.size(), SMatrix::Identity);
for (U32 i = 0; i < STATIC_U32(posedNodes.size()); i++)
{
    SSkeletalPosedNode& posedNode = posedNodes[i];
	
    ...

    const SSkeletalMeshBone& bone = meshAsset->BindPoseBones[boneIndex];
    component->Bones[boneIndex] = bone.InverseBindPoseTransform * posedNode.GlobalTransform;
}

...
```

</details>
{::options parse_block_html="false" /}

For the first game project we're aiming to at the very least implement IK using the <a href="https://www.researchgate.net/publication/220632147_FABRIK_A_fast_iterative_solver_for_the_Inverse_Kinematics_problem">FABRIK</a> solver, 
as well as drive audio through events triggered on specific keyframes in the animation, similar to how Unreal Engine does it with their <a href="https://dev.epicgames.com/documentation/en-us/unreal-engine/animation-notifications-notifies?application_version=4.27">Animation Notifications</a>. 
I'm thinking that triggering these events is best handled when reading the local pose of running animations, as we interpolate position, rotation and scale between keyframes there anyway.

For the future we'd also like to support blend spaces and explore skeletal deformations, to say nothing about building an animation GUI (preferrably a graph utilizing our <a href="{{site.baseurl}}/visual-scripting/">visual scripting</a> system) that's simple to work with.
