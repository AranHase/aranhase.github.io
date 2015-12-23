---
layout: post
title: Tiling a Sprite in Unreal Engine 4
tags: gamedev, cpp, unreal engine 4, ue4, graphics
---

Here is what we will create today:

<iframe width="560" height="315" src="https://www.youtube.com/embed/wCK-Yp7nV9k" frameborder="0" allowfullscreen></iframe>

Tiling a Sprite in Unreal Engine 4.


Setting up our material
-----------------------

First, copy the `DefaultSpriteMaterial` from the engine to a folder in your project. It's inside the `Paper2D Content` in the Engine:

![default spriet](/assets/images/ue4_tiling/TilingMaterialDefault.png)

Then modify this material like below: (I'm naming mine `TilingSpriteMaterial`)

![tilingmaterial](/assets/images/ue4_tiling/TilingSpriteMaterial.png)


The material above has two parameters. `Scale` controls the ratio between world units to texture units. `Translation`
shifts the sampling points.


Setting up our Sprite
---------------------

Just open the sprite and select our new material:


![tilingmaterialinsprite](/assets/images/ue4_tiling/TilingMaterialInSprite.png)

Simple as that. In the video I'm actually using an instance instead of the material, so I can modify the parameters.
In your game you can use a Dynamic Material to adjust those parameters.


## Closing Notes

 * Make sure your texture is set to `Wrap` as TilingMethod in X and Y
 * This was done with UE4 version 4.10.1
