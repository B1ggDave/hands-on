# Common Rendering and JVM

It is always good to question things – do we really need the
JVM backend for the project? What if we could
run the [Mandelbrot Set](https://en.wikipedia.org/wiki/Mandelbrot_set)
rendering on the client? 

Let's implement client-side rendering and see what happens.

## Sharing Code

We've already added our first Kotlin file to the `commonMain`
source set. Let's move all the files from the `jvm` source set
to the `common` source set, so we can share the code with JavaScript.

Wait just a second... It won't work this way, because we are using JVM APIs
to render the images in our code. We need to pick another API to use for
the browser Kotlin/JS part.
We will have to instead use a
[HTML Canvas](https://www.w3schools.com/html/html5_canvas.asp)
to render our images in JavaScript.

It is easy to share code between platforms with Kotlin. 
[Kotlin Multiplatform](https://kotlinlang.org/docs/reference/multiplatform.html)
includes the `expect` and `actual` keywords especially for this. 
We will use the `expect` keyword in the `commonMain` sources to specify
the declarations that we can only implement for a given target.
The `actual` keyword binds the `expect`-ed declarations with their
platform specific counterparts.

Let's have a look at how it works.

## Gradle Dependency

Before we proceed with moving any code, we should include another Gradle
dependency for the `commonMain` source set. Let's add a few more lines
to the `build.gradle.kts` for this:

```kotlin
kotlin.sourceSets["commonMain"].dependencies {
  implementation(kotlin("stdlib-common"))
  implementation("org.jetbrains.kotlinx:kotlinx-html-common:0.6.12")
}
```

This code adds a dependency from the Kotlin standard library. In addition
to this we will include a dependency from the
[kotlinx.html](https://github.com/Kotlin/kotlinx.html)
common library. It adds ways to re-use the HTML, rendering
code between our JVM server-side and JS client-side.

We should refresh the Gradle project model to make sure
the IDE sees all our project model changes. Let's click on the _refresh_
icon in the Gradle tab to accomplish this.

## Moving Code

Sharing code can be done with the following algorithm

* Move all the source files into the `src/commonMain/kotlin` folder
* Replace the missing platform-specific symbols with `expect` declarations
* Add the `actual` keyword for the `expect` declarations for all the platforms

Now it is time to run the algorithm on our project. We need to move
the following files from the `src/jvmMain/kotlin` into `src/commonMain/kotlin`:

* `colour.kt`
* `complex.kt`
* `geometry.kt`
* `render.kt`

We do not need to move the 'main.kt' or `graphics.kt` files, because these files are
platform specific. 'main.kt' starts the HTTP server, which we do not need
on the client-side and the 'graphics.kt` file uses AWT to render the image.
     
There are several errors in the `colour.kt` file. We are missing the
`java.awt.Color` class definition. Let's fix it by adding the `expect`
declarations for colors in `src/commonMain/kotlin/common.kt`:

```kotlin
object Colors

expect class Color
expect fun Colors.newColor(r: Int, g: Int, b: Int): Color
expect val Colors.BLACK : Color
```

The `expect class`, `expect fun`, and `expect val` declarations in the Kotlin/Common
code, instructs the compiler that we'd like to use these declarations in our Kotlin/Common
code, but we are not able to provide implementations. Implementations are provided via the
`actual` keyword independently for the JVM and JS targets (in general,
[Kotlin/Native](https://kotlinlang.org/docs/reference/native-overview.html) can
also be [used](https://kotlinlang.org/docs/tutorials/native/mpp-ios-android.html)
to target iOS and other platforms).

Now that we've fixed the code in the `colour.kt` file to use our expected `Color` class
and the `Colors` object to deal with colors (instead of using the `java.awt.Color` class).
We need to remove import from the `colour.kt` and use the `Colors.newColor`
instead of the `Color` constructor call.

The last fix needs to be done in the `render.kt` file. We need to remove the
import of the `java.awt.Color` class and switch to the `Colors.BLACK`
property in the middle of the file.

## JVM Implementation

We've just fixed the code in the `commonMain` source set using `expect` declarations.
This means that every target (e.g. JVM or JS) has to provide the `actual` declarations
for every `expect` declaration in the `commonMain` file. Let's fix the `jvmMain`
source set first.

We will need to re-create the `colour-actual.kt` file under the `src/jvmMain/kotlin` folder
with the following `actual` declarations:

```kotlin
actual typealias Color = java.awt.Color

actual fun Colors.newColor(r: Int, g: Int, b: Int) = Color(r, g, b)
actual val Colors.BLACK: Color get() = Color.BLACK
```
 
By using these declarations we are saying that our expected class `Color`
should be a `java.awt.Color` class from the JVM. We also provide implementations for
two more required functions.

We should be able to start the JVM application now. Let's check
it by running the Gradle `run` task. We should see something similar to this output
in the console:

```
The Mandelbrot renderer is started at http://127.0.0.1:8888

Open the following in a browser
  http://127.0.0.1:8888/mandelbrot

For a zoomed image try
  http://127.0.0.1:8888/mandelbrot?top=-0.32&left=-0.72&bottom=-0.28&right=-0.68
  http://127.0.0.1:8888/mandelbrot?top=-0.7&left=0.1&bottom=-0.6&right=0.2

```

## Completed Code

We can use the `step-005` branch of the
[github.com/kotlin-hands-on/intro-mpp](https://github.com/kotlin-hands-on/intro-mpp)
repository for the solution to the tasks we did above. 
We can also download the
[archive](https://github.com/kotlin-hands-on/intro-mpp/archive/step-005.zip)
from GitHub directly.
  
