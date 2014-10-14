---
title: 'Design Diary  Shiva3D Screen Boundary Detection in 2 Steps'
author: error454
layout: post
permalink: /2011/08/24/design-diary-shiva3d-screen-boundary-detection-in-2-steps/
categories:
  - Shiva 3D
tags:
  - 2d
  - 3d
  - design
  - game
  - indie
  - screen boundary
  - shiva
  - shiva3d
---
In a <a href="http://mobilecoder.wordpress.com/2011/07/07/design-diary-shiva-3d-camera-implementation-for-2d-in-3d/" target="_blank">previous design diary</a>, I talked about screen boundary detection.  What is an easy way to determine when an object has left the screen so that you can act on it?

I discovered a new solution  that is incredibly cheap and easy with a few caveats:

*   You will not know which edge of the screen the object has passed (so it won't work for wrapping objects around on the screen)
*   You only get one notification when the object moves beyond the camera (unless you setup a timer)



## How to Implement in 2 Steps

<strong>1</strong>. For each model that you want to get boundary notifications for, open the model properties and under general, check Frustum Activation.

<img src='{{ site.url }}/assets/uploads/2011/08/frustum.jpg' alt=''>

**2**. In one of the AI models attached to your object, implement the onDeactivate handler.

<a href='{{ site.url }}/assets/uploads/2011/08/frustum.jpg'><img src='{{ site.url }}/assets/uploads/2011/08/ondeactivate.jpg' alt=''></a>

Now any time an object with this AI leaves the screen, onDeactivate will be called.  Similarly, onActivate is called when the object enters the screen.  For us, this made destroying rockets at the screen boundary as easy as:

<pre>--------------------------------------------------------------------------------
function whistlePeteAI.onDeactivate (  )
--------------------------------------------------------------------------------

	scene.destroyRuntimeObject ( application.getCurrentUserScene ( ), this.getObject ( ) )

--------------------------------------------------------------------------------
end
--------------------------------------------------------------------------------
</pre>