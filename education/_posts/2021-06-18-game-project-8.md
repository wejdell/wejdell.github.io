---
layout: post
title: Game Project 8 - Besöket
perma-link: "/education/project-8/"
featured-image: /assets/images/project-8.gif
tags: [project, deferred rendering, rendering, ui, vfx, audio]
date-string: JUNE 18, 2021
---

## Specifications

* **Genre**:    Horror FPX
* **Duration**: 9 Weeks Full-Time, 8h/day
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

