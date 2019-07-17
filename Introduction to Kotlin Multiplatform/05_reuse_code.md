# Avoiding Code Duplication

Right now we have code duplication in our project. We've just hard-coded
the URL of the JVM backend in the JS frontend. In the `src/jsMain/kotlin/main.kt`
we have the following lines

```kotlin
const val jvmBackend = "http://127.0.0.1:8888"
```

In the `src/jvmMain/kotlin/main.kt` we have:

```kotlin
fun main() {
  val host = "127.0.0.1"
  val port = 8888

  val server = embeddedServer(Netty, host = host, port = port) //...
```

## Using Kotlin/Common

The [kotlin-multiplatform](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html) plugin
supports Kotlin/Common code and is a way to share code
between different targets. Let's use it to avoid duplication
of the JVM server address. 

First, we need to create a common file for the project with a
`src/commonMain/kotlin/common.kt` path and the following contents

```kotlin
const val jvmHost = "127.0.0.1"
const val jvmPort = 8888
``` 

The declarations from the Kotlin files under
`src/commonMain/kotlin` are visible to all the targets. We can only
use a platform-independent subset of the Kotlin standard library
from Kotlin/Common source files. Kotlin Multiplatform libraries are allowed too.
It allows us to easily share code between platforms.

Let's update the code in both the `jsMain` and `jvmMain` source
sets to use the newly added `jvmHost` and `jvmPort` constants.

## Using Common Code

Let's use the new constants in the JS code from the `src/jsMain/kotlin/main.kt`

```kotlin
const val jvmBackend = "http://$jvmHost:$jvmPort"
```

In the `src/jvmMain/kotlin/main.kt` we should have

```kotlin
fun main() {
  val host = jvmHost
  val port = jvmPort

  val server = embeddedServer(Netty, host = host, port = port) //...
```

We can inline the `host` and `port` variables to simplify the code above.

## Running the App

Now we can start both the server-side JVM process,
by running the `run` task in Gradle, and the client-side by running the `jsRun` task (please note, it may
not work on Windows operating systems due to a known bug).

## Completed Code

We can use the `step-004` branch of the
[github.com/kotlin-hands-on/intro-mpp](https://github.com/kotlin-hands-on/intro-mpp)
repository for a solution to the tasks we did above. 
We can also download the
[archive](https://github.com/kotlin-hands-on/intro-mpp/archive/step-004.zip)
from GitHub directly.
  
