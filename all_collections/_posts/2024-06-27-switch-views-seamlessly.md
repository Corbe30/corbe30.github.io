---
layout: post
title: Seamless View Switching Algorithm for ArcRotateCamera in Babylon.js
date: 2024-06-27 10:00:00
categories: [babylonjs, algorithm]
---

I'll describe how I derived an algorithm to switch seamlessly between perspective and orthographic views. The solution will be implemented in babylon.js.

## Basics (the why)
A camera can be controlled by 3 operations - pan, zoom and orbit.
These operations in arcRotateCamera are performed by changing the following properties:
1. camera position (where the camera is at)
2. camera target (where the camera is looking at)

More about how it works [here](https://doc.babylonjs.com/features/featuresDeepDive/cameras/camera_introduction#arc-rotate-camera).

Consider using only these two properties to perform the operations. Although pan and orbit changes are reflect in both the views, zoom doesn't seem to have any effect in orthographic view (see [why?](https://blenderartists.org/t/zooming-through-orthographic-cameras-not-working-need-everyones-help-to-fix-this/701450/3))

So we need a third parameter to control zooming in ortho view - which is `visible area`. Instead of going back and forth, we make the camera's viewing area larger or smaller. In babylon.js, this is done by modifying ortholeft, orthoright, orthobottom and orthotop properties. 

Using this solution is fine in orthographic view, but once we try to apply it in perpective view, the edges and corners of your view may appear to bend outwards. Think of it as changing from 1x to 0.5x in your phone's camera. To zoom out, you'd rather want to move your phone back, than switch to 0.5x zoom.

> If you are adamant, in perspective view, `fov` is used to expand the boundaries. In ortho view, [the fov is actually 0](https://gamedev.stackexchange.com/a/64431).

So now, to handle zooming, we need a function which converts ortho-values (for orthographic view) to target (for perspective view) and visa versa.


## Algorithm
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. 

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require("./lang/" + l);
  return true;
};
```

Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.