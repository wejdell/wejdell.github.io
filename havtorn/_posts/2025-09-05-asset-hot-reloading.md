---
layout: post
title: "Asset Hot Reloading"
perma-link: "/havtorn/asset_hot_reload/"
featured-image: /assets/images/assetHotReload.gif
tags: [havtorn, editor, assets]
---

The asset registry in Havtorn was recently refactored to move all asset data (e.g. vertices in a mesh, the pixels of a texture) to a centralized location. Instead of each ECS component having its own copy of the asset data it needs to reference, as before,
every component now has one or more `AssetReference`s. These are easy to serialize, and we can let the asset registry have responsibility over the lifetime of the asset data. Systems working on entities just request the data from the registry using the asset references
held by components. 

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>

```c++
struct SAssetReference
{
    U32 UID = 0;
    std::string FilePath = "NullAsset";
    
    SAssetReference() = default;
    explicit SAssetReference(const std::string& filePath)
    {
        FilePath = UGeneralUtils::ConvertToPlatformAgnosticPath(filePath);
        U32 prime = 0x1000193;
        UID = 0x811c9dc5;
        
        for (U64 i = 0; i < FilePath.size(); ++i)
        {
            U8 value = FilePath[i];
            UID = UID ^ value;
            UID *= prime;
        }
    }
    
       ...
}
```
</details>
{::options parse_block_html="false" /}

<!--excerpt-begin-->
The gif above shows the first iteration of hot reloading assets, where for any asset that was originally authored in an external program such as Blender or Photoshop, the registry can dynamically reimport it as soon as the source file (fbx, dds) changes on disk.
 As an example, a mesh can be placed in the editor, then the source file watching on the asset be toggled, and whenever the fbx is changed on disk the engine reimports it with the same settings on the fly.
<!--excerpt-end-->
We don't need to update all the components that care about that particular asset, the registry just internally updates the data that it manages.

The only reason that enabling the file watching is a manual step is so that we don't hog resources by watching for file changes when it's not needed. Of course a more automatic setup could be more fitting for exploratory phases of a project.

## Asset Redirection

As part of this I also implemented rudimentary *asset redirection*, somewhat similar to what Unreal Engine does, meaning that assets can be moved around in the editor to different directories, and a link is created between the old file path and the new. 
If an asset is already loaded, the components using it can continue to do so with the same `AssetReference`. If the asset *isn't* loaded, we traverse the redirections to try to find an existing file. If we do, we continue loading as necessary.

{::options parse_block_html="true" /}
<details><summary markdown="span">**View Code**</summary>

```c++
bool CAssetRegistry::LoadAsset(const SAssetReference& assetRef)
{
    std::string filePath = assetRef.FilePath;
    if (!UFileSystem::Exists(filePath))
    {
        CJsonDocument config = UFileSystem::OpenJson(UFileSystem::EngineConfig);
        std::string redirection = config.GetValueFromArray("Asset Redirectors", filePath, "");
                        
        while (!UFileSystem::Exists(redirection) && redirection != "")
        {
            redirection = config.GetValueFromArray("Asset Redirectors", redirection, "");
        } 
            
        if (redirection == "")
        {
            HV_LOG_WARN("CAssetRegistry::LoadAsset: Asset file pointed to by %s failed to load, does not exist!", assetRef.FilePath.c_str());
            return false;
        }
            
        filePath = redirection;
    }
    
    // Continue with asset loading
    ...
}
```

</details>
{::options parse_block_html="false" /}

We can abuse the fact that the `AssetReference`s are just file path strings hashed into identifiers on construction. When an asset has been redirected and we succesfully load it, 
we can keep the hashed identifier we got from the original file path, instead of updating the identifier to correspond to the redirected path. That means that every following requester (other
components wanting to use the same redirected asset) can immediately get a hold of the asset with the identifier they already have stored. We only need to resolve the redirection once per load of the asset.

There's no automatic clean up of the redirectors yet, but there is a manual step where you can click a button to update all assets on disk, if they have dependencies on other assets. Then you would have to go through
all components in all scenes (where ECS entities and components are stored) and update the asset they are pointing to. I think an automatic solution is doable though, you would just have to go through all entities on disk, and iterate through all components to see if
there are any old asset references.
