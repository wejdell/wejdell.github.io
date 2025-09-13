---
layout: post
title: "Asset Hot Reloading"
perma-link: "/havtorn/asset_hot_reload/"
featured-image: /assets/images/assetHotReload.gif
tags: [havtorn, editor, assets]
date-string: SEPTEMBER 05, 2025
---

<!--excerpt-begin-->
This is a first version of hot reloading assets, meaning for any asset that was authored in an external program such as Blender or Photoshop, the asset system in Havtorn can dynamically reimport any asset as soon as the source file (fbx, dds) changes on disk.
As an example, a mesh can be placed in the editor, then the source file watching on the asset toggled, and whenever the fbx is changed on disk the engine reimports it with the same settings on the fly.
<!--excerpt-end-->

The only reason that enabling the file watching is a manual step is so that we don't hog resources by watching for file changes when it's not needed. If preferred, we may pay the price to automatically enable watching of all currently loaded assets instead.

As part of this I also implemented rudimentary *asset redirection*, somewhat similar to what Unreal Engine does, meaning that assets can be moved around in the editor to different directories, and a link is created between the old file path and the new. 
There are ways to clean these up fully with a bit of manual labor in the editor.