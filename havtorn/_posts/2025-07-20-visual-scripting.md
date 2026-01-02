---
layout: post
title: "Visual Scripting"
perma-link: "/havtorn/visual_scripting/"
featured-image: /assets/images/visual_scripting.gif
tags: [havtorn, editor, scripting]
---

Havtorn uses its own visual scripting system, tailored to the custom ECS I built. The graphical node rendering is based on thedmd's <a href="https://github.com/thedmd/imgui-node-editor">imgui node editor</a>.

<!--excerpt-begin-->
In the example we see a physics trigger getting a ScriptComponent referencing the test script asset we authored in the separate script editor. 
 When the block suspended in the air above overlaps with the trigger volume, we change the mesh from a block to the clock mesh, and the green point light in the foreground is toggled off. This was set up to show interactions between a few separate entities, our so called data bindings, and a non-trivial entry point for the script (OnBeginOverlap, instead of a simple Tick for example).
<!--excerpt-end-->

Scripts are thought of as ECS systems, modifiable in editor on an asset level. They are not the same as, e.g., the editor representations of Unreal Engine's objects. Because they are assets (like a model mesh or animation clip), only one instance can be loaded at a time.
They way they then work with persistent data, is by *declaring* what data they are expecting to work on, and receiving that data from each ECS component (holder of data) that *defines* it. For lack of a better term, I called this concept a *data binding* between the script and component.

In the example, the work that the script is meant to perform when triggered, is to shut off a point light on an ECS entity, and change the static mesh of the entity triggering the physics trigger volume. It can shut off any point light, and change the mesh to any mesh. We see in the clip 
that the script just declares its data bindings, and then we define exactly what point light entity to toggle the light on, and what mesh to set, in the component referencing the script. The data itself is owned by the component, but the script declares what it should be.

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>
```c++
// CScriptSystem.cpp
...

for (SScriptComponent* component : scriptComponents)
{
    ...
    
    SScript* script = GEngine::GetAssetRegistry()->RequestAssetData<SScript>(component->AssetReference, component->Owner.GUID);
    
    if (component->DataBindings.size() != script->DataBindings.size())
        component->DataBindings.resize(script->DataBindings.size());
    
    // Input component values before running script
    for (U64 i = 0; i < script->DataBindings.size(); i++)
    {
        SScriptDataBinding& scriptBinding = script->DataBindings[i];
        const SScriptDataBinding& componentBinding = component->DataBindings[i];
        
        scriptBinding.Data = componentBinding.Data;
    }
    
    ...
    
    // Start traversing by some means
    if (beganPlay && script->HasNode(BeginPlayNodeID))
        script->TraverseFromNode(BeginPlayNodeID, scene.get());
    
    if (script->HasNode(TickNodeID))
        script->TraverseFromNode(TickNodeID, scene.get());
    
    if (endedPlay && script->HasNode(EndPlayNodeID))
        script->TraverseFromNode(EndPlayNodeID, scene.get());
    
    // Output script values to store in component after running script
    for (U64 i = 0; i < script->DataBindings.size(); i++)
    {
        const SScriptDataBinding& scriptBinding = script->DataBindings[i];
        SScriptDataBinding& componentBinding = component->DataBindings[i];
        
        componentBinding.Data = scriptBinding.Data;
    }
}

...
```
</details>
{::options parse_block_html="false" /}

This means that any number of entities can reference a script (loaded once in memory) and have it run logic on the entity's own data. Just like any other ECS system, the script's job is to 
manipulate data owned by ECS components. It's just a system that you can author in the editor.

## Authoring Nodes

Havtorn doesn't have a reflection system (yet?), and authoring nodes is a manual process currently. The overhead isn't huge, but we're considering what the best strategic approach would be. Perhaps that would be having many small nodes that affect one property, and try to streamline the creation of those, or focus more on larger nodes that have more control over all the data held in a component for example.
I think it will have to be some sort of combination, and should be driven by the needs of the current project we're working on. Here's an example of a simple core functionality node.

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>
```c++

// CoreNodes.h
struct SBranchNode : public SNode
{
    ENGINE_API SBranchNode(const U64 id, const U32 typeID, SScript* owningScript);
    virtual ENGINE_API I8 OnExecute() override;
};

// CoreNodes.cpp
SBranchNode::SBranchNode(const U64 id, const U32 typeID, SScript* owningScript)
    : SNode::SNode(id, typeID, owningScript, ENodeType::Standard)
{
    AddInput(UGUIDManager::Generate(), EPinType::Flow);
    AddInput(UGUIDManager::Generate(), EPinType::Bool, "Condition");

    AddOutput(UGUIDManager::Generate(), EPinType::Flow, "True");
    AddOutput(UGUIDManager::Generate(), EPinType::Flow, "False");
}

I8 SBranchNode::OnExecute()
{
    bool condition = false;
    GetDataOnPin(EPinDirection::Input, 1, condition);

    // Return pin index to continue traversal from
    return condition ? 0 : 1;
}
```
</details>
{::options parse_block_html="false" /}

<div class="click-zoom">
    <label>
        <input type="checkbox" />
        <img class="centered" src="/assets/images/branch-node.png">
    </label>
</div>