# Project Description

In this tutorial, we will look at how to create a Kotlin multiplatform project
and set up code reuse between a client-side JavaScript application and a
server-side JVM applications. Throughout the tutorial, we will
use the pre-implemented
[Mandelbrot Set](https://en.wikipedia.org/wiki/Mandelbrot_set)
rendering algorithm and run it for both 
server-side and client-side
image rendering.

We'll see how our JVM-only implementation can be turned into a
fully functional Kotlin multiplatform project compiled for both
JVM and JavaScript platforms.


## Basics

Fractals and the [Mandelbrot Set](https://en.wikipedia.org/wiki/Mandelbrot_set)
are good examples of computer science class assignments. All we need to know right now
is that every pixel in the image is mapped to a 2D coordinate
(e.g. `0.03, 0.045`), we run slow computations for every pixel to figure out
the color of them, and the visible area of the picture
is `[-2.0 .. 2.0] x [-2.0 .. 2.0]`.

![](./assets/mandelbrot-full.png)

The Mandelbrot set is a fractal or what's sometimes referred to as a self-repeating structure. As we zoom in,
we're going to see something similar to what we've seen before. It
opens up oppurtunities for fractal rendering tools -- it should
support a zoom feature to make investigating fractals easier. 

![](./assets/mandelbrot-zoom1.png)

or 

![](./assets/mandelbrot-zoom2.png)
