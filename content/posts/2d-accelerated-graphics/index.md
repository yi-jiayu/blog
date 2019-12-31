---
title: "2D accelerated rendering with SDL in Chicken Scheme"
subtitle: "Using the sdl2 egg with the C API for reference"
date: 2019-05-09 18:20:43+08:00
tags: ["scheme", "chicken", "graphics", "lisp"]
---

After my semi-successful attempt at writing a terminal-based CHIP-8 intepreter in Go (more about it
in a separate post), I've decided to try again using [Chicken Scheme](https://www.call-cc.org/) and
[SDL](https://www.libsdl.org/), mostly as an excuse to pick up another Lisp dialect and to escape
from the limitations of the terminal (mainly the lack of keyup events).

## Using SDL in Chicken Scheme

The [sdl2](http://wiki.call-cc.org/eggref/5/sdl2) egg provides bindings to the SDL 2 API in Chicken
Scheme, and its source code includes some usage examples.
[`demos/basic.scm`](https://gitlab.com/chicken-sdl2/chicken-sdl2/blob/master/demos/basics.scm)
demonstrates how to create an SDL window and move two smiley faces around with the keyboard and
mouse:

{{< figure src="demo.png" alt="The window created by running demos/basic.scm" class="nooutline" >}}

The demo works by [creating surfaces][1] and using them to [update the window surface][2]. Besides
surfaces, SDL also has textures, and according to [this Stack Overflow
answer](https://stackoverflow.com/a/26113388), the main difference between the two is that surfaces
are used in software (or CPU) rendering, while textures are used in hardware (GPU) rendering, and so
rendering textures is much faster than rendering surfaces.

## 2D accelerated rendering

After some more research, I came across this useful post which contained a step-by-step walkthrough
together with examples of using 2D accelerated rendering in SDL 2!

{{< linkpreview title="Using SDL2: 2D Accelerated Renderering"
description="Psst! Do you want to know a way to get your SDL2 applications running faster?"
url="https://dev.to/noah11012/using-sdl2-2d-accelerated-renderering-1kcb" >}}

Let's use it to modify the `basics.scm` demo to use 2D accelerated rendering!

{{< linkpreview title="demos/basics.scm"
description="25d66ebd4383a9847bd26c9f69dbfc0e54e2a8b4 · chicken-sdl2 / chicken-sdl2 · GitLab"
url="https://gitlab.com/chicken-sdl2/chicken-sdl2/blob/25d66ebd4383a9847bd26c9f69dbfc0e54e2a8b4/demos/basics.scm" >}}

One slight complication: the post uses the SDL C API, while we want to do it in Chicken Scheme.
Fortunately, it turns out that's hardly an issue---the `sdl2` egg closely follows the C API, and
it's not difficult to translate the C code into Chicken Scheme.

## Modifying `basics.scm`

The actual rendering to screen in `basics.scm` happens in the `draw-scene!` procedure at [line
158](https://gitlab.com/chicken-sdl2/chicken-sdl2/blob/25d66ebd4383a9847bd26c9f69dbfc0e54e2a8b4/demos/basics.scm#L158) (note that `sdl2` module functions are called with a `sdl2:` prefix, as recommended in the
 [documentation](http://wiki.call-cc.org/eggref/5/sdl2#usage-and-examples)):

```scheme
(define (draw-scene!)
  (let ((window-surf (sdl2:window-surface window)))
    ;; Clear the whole screen using a blue background color
    (sdl2:fill-rect! window-surf #f (sdl2:make-color 0 80 160))
    ;; Draw the smileys
    (draw-obj! smiley2 window-surf)
    (draw-obj! smiley1 window-surf)
    ;; Refresh the screen
    (sdl2:update-window-surface! window)))
```

We just need to modify this function to use 2D accelerated rendering.

First, we need to create a `SDL_Renderer`: 

In C:
```c
SDL_Window *window = ...;

SDL_Renderer *window_renderer = SDL_CreateRenderer(window, -1, SDL_RENDERER_ACCELERATED);
```

In `basics.scm`, our window object is created on [line 63][3]:

```scheme
;;; Create a new window.
(define window
  (sdl2:create-window!
   "SDL Basics"                         ; title
   'centered  100                       ; x, y
   800  600                             ; w, h
   '(shown resizable)))                 ; flags
```

We use the [`create-renderer!`](https://wiki.call-cc.org/eggref/5/sdl2#renderer-functions) procedure to create a new renderer:

```scheme
(define renderer (sdl2:create-renderer! window -1 (list 'accelerated)))
```

The linked blog post writes "don't try to get the window's surface if you also want to use the
window's renderer. Bad things will happen if you do", so let's first modify `draw-scene!` to avoid
using the window surface and create a new one instead with
[`make-surface*`](http://wiki.call-cc.org/eggref/5/sdl2#sdl2surface):

```diff
 (define (draw-scene!)
-  (let ((window-surf (sdl2:window-surface window)))
-    ;; Clear the whole screen using a blue background color
-    (sdl2:fill-rect! window-surf #f (sdl2:make-color 0 80 160))
+  ;; Create a new surface the same size as the window
+  (let ((surf (sdl2:make-surface* 800 600 32)))
+    ;; Clear the new surface screen using a blue background color
+    (sdl2:fill-rect! surf #f (sdl2:make-color 0 80 160))
     ;; Draw the smileys
-    (draw-obj! smiley2 window-surf)
-    (draw-obj! smiley1 window-surf)
+    (draw-obj! smiley2 surf)
+    (draw-obj! smiley1 surf)
```

Note that we're using the `make-surface*` variant with an asterisk, which means that the created
object is not managed by the Chicken Scheme garbage collector, so we'll have to make sure to free it
later with `free-surface!` ([Struct Memory
Management](http://wiki.call-cc.org/eggref/5/sdl2#struct-memory-management)).

Next, we need to convert our `SDL_Surface` into a `SDL_Texture`.

In C:

```c
SDL_Surface *surface = ...;

SDL_Texture *texture = SDL_CreateTextureFromSurface(window_renderer, surface);

if(!texture)
{
    std::cout << "Failed to convert surface into a texture\n";
    std::cout << "SDL2 Error: " << SDL_GetError() << "\n";
}

SDL_FreeSurface(surface);
```

Before we update the screen, we'll do the conversion using
[`create-texture-from-surface*`](http://wiki.call-cc.org/eggref/5/sdl2#sdl2texture) and then free
the surface:

```diff
     (draw-obj! smiley2 surf)
     (draw-obj! smiley1 surf)
+    ;; Convert the surface to a texture
+    (let ((texture (sdl2:create-texture-from-surface* renderer surf)))
+      ;; TODO: check if the texture creation was successful
+      ;; Free the surface since we're done converting it to a texture
+      (sdl2:free-surface! surf))
```

Finally, we have to clear the screen, copy our texture into the renderer and then render it to the
screen.

In C:

```c
SDL_RenderClear(window_renderer);

SDL_RenderCopy(window_renderer, texture, nullptr, nullptr);

SDL_RenderPresent(window_renderer);
```

The corresponding Chicken functions are `render-clear!`, `render-copy!` and `render-present!`. We'll
also free the texture with `destroy-texture!`:

```diff
       (sdl2:free-surface! surf))
+      ;; Clear the screen
+      (sdl2:render-clear! renderer)
+      ;; Copy our texture into the renderer
+      (sdl2:render-copy! renderer texture)
+      ;; Render to the screen
+      (sdl2:render-present! renderer)
+      ;; Destroy the texture after we're done
+      (sdl2:destroy-texture! texture))))
```

That's it! The final, modified `draw-scene!`, with the `create-renderer!` call as well:

```scheme
(define renderer (sdl2:create-renderer! window -1 (list 'accelerated)))

(define (draw-scene!)
  ;; Create a new surface the same size as the window
  (let ((surf (sdl2:make-surface* 800 600 32)))
    ;; Clear the new surface screen using a blue background color
    (sdl2:fill-rect! surf #f (sdl2:make-color 0 80 160))
    ;; Draw the smileys
    (draw-obj! smiley2 surf)
    (draw-obj! smiley1 surf)
    ;; Convert the surface to a texture
    (let ((texture (sdl2:create-texture-from-surface* renderer surf)))
      ;; TODO: check if the texture creation was successful
      ;; Free the surface since we're done converting it to a texture
      (sdl2:free-surface! surf)
      ;; Clear the screen
      (sdl2:render-clear! renderer)
      ;; Copy our texture into the renderer
      (sdl2:render-copy! renderer texture)
      (sdl2:render-present! renderer)
      ;; Destroy the texture after we're done
      (sdl2:destroy-texture! texture))))
```

## Finishing up

If we recompile `basics.scm` now and run it again, we'll see...

Absolutely no difference (at least I didn't)!

I don't think hardware acceleration actually has much of an impact for such a simple example, but it
doesn't seem too difficult to implement, so I'm still looking forward to using it in my emulator \o/.

Here's the full diff of the changes I made as a GitHub Gist:

{{< linkpreview title="yi-jiayu/basics-2d-accelerated.patch"
description="Modifying the Chicken Scheme sdl2 egg basics.scm demo to use 2D Accelerated Rendering"
url="https://gist.github.com/yi-jiayu/7b3a2506f7bf66d721992456295a1604" >}}

[1]: https://gitlab.com/chicken-sdl2/chicken-sdl2/blob/25d66ebd4383a9847bd26c9f69dbfc0e54e2a8b4/demos/basics.scm#L117
[2]: https://gitlab.com/chicken-sdl2/chicken-sdl2/blob/25d66ebd4383a9847bd26c9f69dbfc0e54e2a8b4/demos/basics.scm#L166
[3]: https://gitlab.com/chicken-sdl2/chicken-sdl2/blob/25d66ebd4383a9847bd26c9f69dbfc0e54e2a8b4/demos/basics.scm#L63
