---
layout: post
title: Unreal Engine 4 Transition Screens
tags: gamedev, cpp
---


Here is what we will create today:

![sphere_mask][1]

Setting up our material
-----------------------

The main part of this effect is in the transition material. We will be using a
"User Interface" material because it will be placed on a ``User Widget`` later.
The same effect could be made as a `Post Process`, but that method will not be
covered here.

The transition effect uses a grayscale image combined with a threshould to
determine what parts of an image to make opaque and what parts to make
transparent. The threshould is being set using a ``Scalar Parameter`` on a
``Dynamic Material`` ([Learn more
here](https://docs.unrealengine.com/latest/INT/Engine/Rendering/Materials/MaterialInstances/index.html)).
Let's see the solution:

![sphere_mask][2]

**Explanation:** The parts of the image with a pixel intensity higher then the
threshould will be made transparent, while the rest will be full opaque. For
that `SphereMask`, where the center is more bright than the rest of the image,
the center will be the first part to disappear as the threshould decreases. The
corners will be the last things to disappear.


And that is it for the material, next we will put it on an UI.


Setting up our UI Widget
------------------------

We want to cover the transition effect over the entire screen. For that, create
a new `User Widget` and create an image that covers the entire canvas, and just
make sure the `anchoring` behaviour is correct, as in:

![ui][3]

Then, when the widget begins play, create our dynamic material and set the
image to use it:

![ui][4]

We will then start the transition effects when the `Close` custom method is
called:

![ui][5]

The `Tick` method will only decrease the `Mask Threshould` on our dynamic
material. Once the thershould hits zero, the widget will be removed from view.
I added a `Transition Speed` variable that controls how fast the threshould
will decrease. Adjust it for your liking.

That is it for the UI, now we need to add the UI to our player's camera.


Showing the transition effect on screen
---------------------------------------

For simplicity, I'll just use the Twin Stick Shooter template that comes with
Unreal, and will create our transition effect when our Pawn is instanciated.

![pawn][6]

I'm using a delay of one second before closing the transition effect to
simulate a loading screen.

And that is it. I'll leave some extra examples...


Some extra examples
-------------------

### Noise Mask

Using a noise material node:

![noise][8]

![noise][7]


### Texture Mask

Or using a texture: (download the texture [`.png`][11] or the [`Inkscape file`][12])

![texture][10]

![texture][9]

## Closing Notes

 * Others UI elements may show on top of your transition effect
 * It is possible to do the same thing using Post Process Blendables, but not sure what advantages there are for it


 [1]: /assets/images/ue4_transitions/transition_sphere.gif
 [2]: /assets/images/ue4_transitions/sphere_mask.png
 [3]: /assets/images/ue4_transitions/ui_widget.png
 [4]: /assets/images/ue4_transitions/ui_widget_begin.png
 [5]: /assets/images/ue4_transitions/ui_widget_tick.png
 [6]: /assets/images/ue4_transitions/pawn_begin.png
 [7]: /assets/images/ue4_transitions/noise_mask.png
 [8]: /assets/images/ue4_transitions/transition_noise.gif
 [9]: /assets/images/ue4_transitions/texture_mask.png
 [10]: /assets/images/ue4_transitions/transition_texture.gif
 [11]: /assets/images/ue4_transitions/transition.png
 [12]: /assets/images/ue4_transitions/transition.svg
