# Common Rendering and JS

We are now ready to update our Kotlin/JS code and provide 
`actual` declarations for all the `expect` declarations from the `commonMain`. 
We will be using the [HTML Canvas](https://www.w3schools.com/html/html5_canvas.asp)
on the client-side to render the images.

## Adding Colors

The actual `Color` class for the client-side can be implemented from scratch.
Let's create a `colour-actual.kt` file in the `src/jsMain/kotlin` folder
with the following contents in it:

```kotlin
actual class Color(
        val r: Int,
        val g: Int,
        val b: Int
)
actual fun Colors.newColor(r: Int, g: Int, b: Int) = Color(r, g, b)
actual val Colors.BLACK: Color get() = Color(0, 0, 0)
```

We've added the `actual` declarations for the `Colors.BLACK` property
and for the `Colors.newColor` function.
This should be enough to fix the compilation of the `jsMain` source set. Now it is
time to support rendering to the HTML Canvas.

## Adding Canvas Support

Let's add the implementation code to render images into the HTML Canvas. We'll need
to create a `src/jsMain/kotlin/canvas.kt` file with the following contents
```kotlin
fun renderToCanvas(canvas : HTMLCanvasElement, action: (FractalImage) -> Unit) {
  val ctx = canvas.getContext("2d") as CanvasRenderingContext2D
  ctx.clearRect(0.0, 0.0, ctx.canvas.width.toDouble(), ctx.canvas.height.toDouble())

  val imageData = ctx.createImageData(
      ctx.canvas.width.toDouble(),
      ctx.canvas.height.toDouble())

  val image = object:FractalImage {
    override val pixelRect
      get() = Rect(
        left = 0, top = 0,
        right = imageData.width, bottom = imageData.height)

    override fun putPixel(p: Pixel, c: Color) {
      val base = 4 * (p.x + imageData.width * p.y)
      val image : dynamic = imageData.data
      image[base + 0] = c.r
      image[base + 1] = c.g
      image[base + 2] = c.b
      image[base + 3] = 255
    }
  }
  action(image)
  ctx.putImageData(imageData, 0.0, 0.0)
}
```

## Using the Canvas

The final step is to call the rendering code from the `main.kt` file. 
Let's add a few lines to the `src/jsMain/kotlin/main.kt` file, so 
the `main()` function adds a canvas element after the `img()` function call:
```kotlin
img//(...)
canvas {
  id = "canvas"
  width = "600"
  height = "600"
}
```

We also add a call to render the code at the end of the `main()` function:

```kotlin
renderToCanvas(document.getElementById("canvas") as HTMLCanvasElement) { image ->
  MandelbrotRender.justRender(300, image, MandelbrotRender.initialArea)
}
```

## Running the client-side App

It's now time to run the Gradle `run` task to start the server-side Kotlin/JVM project and the
`jsRun` task to start the client-side Kotlin/JS application. We should be
able to see two images – the one that is rendered server-side with the Kotlin/JVM, and a second
that is rendered directly in the browser with Kotlin/JS:

![](./assets/site-full.png)


## Completed Code

We can use the `step-006` branch of the
[github.com/kotlin-hands-on/intro-mpp](https://github.com/kotlin-hands-on/intro-mpp)
repository for the solution to the tasks we did above. 
We can also download the
[archive](https://github.com/kotlin-hands-on/intro-mpp/archive/step-006.zip)
from GitHub directly.
   
