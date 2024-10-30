# Writing Gradle plugins in Kotlin

Kotlin has [a great Java interoperability story](https://kotlinlang.org/docs/java-to-kotlin-interop.html), making it a good language to write Gradle plugins.

For complete compatibility, several aspects require extra care. This page gives an overview of the different compatibility issues and recommended setup to avoid them.   

## Gradle compatibility

When executing a build, Gradle [forces its own version of `kotlin-stdlib`](https://github.com/gradle/gradle/issues/16345), the embedded version. 

For this reason, your plugin must depend on a version of `kotlin-stdlib` that is compatible with the embedded version.

Gradle publishes the embedded versions in the [Kotlin compatibility matrix](https://docs.gradle.org/current/userguide/compatibility.html#kotlin).

For an example, at the time of writing, Gradle 8.10 embeds `kotlin-stdlib:1.9.24`.

### Making your code compatible with the Kotlin embedded version

You can use a more recent version of the Kotlin Gradle Plugin, but you'll have to make sure not to call any 2.0 API:

```kotlin
plugins {
  // Use latest version of the Kotlin Gradle Plugin
  id("org.jetbrains.kotlin.jvm").version("2.0.21")
  // java-gradle-plugin creates marker publications and plugin descriptors 
  id("java-gradle-plugin")
}

tasks.withType<KotlinCompile>().configureEach {
  // But make sure your plugin code only uses 1.9 APIs
  compilerOptions.apiVersion.set(KotlinVersion.KOTLIN_1_9)
}

kotlin {
  // Also make sure to depend on 1.9 kotlin-stdlib
  // See also https://youtrack.jetbrains.com/issue/KT-53462
  coreLibrariesVersion = "1.9.24"
}
```

### Ensuring dependencies are compatible with the Kotlin embedded version

In addition to your own code, your dependencies must also use a compatible version of `kotlin-stdlib`.

Because the compiler doesn't run on dependencies, `apiVersion` does not help here, you'll have to check that the dependencies do not depend on a newer version of `kotlin-stdlib`.

This can be done using a custom Gradle task:

```kotlin
/**
 * An example of a task that checks that no kotlin-stdlib > 1.9.24 is pulled
 * in the classpath.
 * Configuration cache and edge cases are left as an exercise to the reader.
 */
tasks.register("checkGradleCompatibility") {
  doLast {
    val root = configurations.getByName("runtimeClasspath").incoming.resolutionResult.rootComponent.get()
    root.dependencies.forEach {
      if (it is ResolvedDependencyResult) {
        val rdr = it
        val requested = rdr.requested
        val selected = rdr.selected
        if (
          requested is ModuleComponentSelector
          && requested.group == "org.jetbrains.kotlin"
          && requested.module == "kotlin-stdlib"
        ) {

          val requestedVersion = requested.version
          val selectedVersion = selected.moduleVersion?.version
          check (selectedVersion == requestedVersion) {
            "kotlin-stdlib was upgraded to $selectedVersion"
          }
        }
      }
    }
  }
} 
```

### Alternative #1: relocating kotlin-stdlib 

If the steps above are too complicated, maybe because a required dependency uses a newer version of Kotlin, or because your own plugin code requires newer Kotlin features, you can shadow a relocated version of `kotlin-stdlib` that doesn't clash with the Gradle embedded one.

To do this, you can use [R8](https://github.com/GradleUp/GR8). You can read more about the process [in this dedicated blog post](https://blog.mbonnin.net/use-latest-kotlin-in-your-gradle-plugins). 

> [!NOTE] [Shadow](https://github.com/GradleUp/shadow/) could be an alternative, but we have found that it doesn't work reliably because [it relocates String constants as well](https://github.com/GradleUp/shadow/issues/232)

### Alternative #2: using separate classloaders

Another solution if you want to use a newer `kotlin-stdlib` without using relocation is to run your code in a separate, isolated, classloader. The glue code of your plugin and initialization still has to be compatible but as soon as you switch to a new classloader, you can use any dependencies without any risk of incompatibilities.

Projects such as [Gratatouille](https://github.com/GradleUp/Gratatouille) can help with that.

## Groovy interoperability

Because your plugin may be used from Groovy build scripts (`build.gradle`), it is important to have Groovy compatibility in mind.

### General interoperability

In general Groovy does not know anything about Kotlin. Avoid Kotlin-only features such as:

- extension functions
- default parameter values
- function types
- receivers
- etc... 

These features may be used in extra functionality for Kotlin callers but it is important that all the base functionality of your plugin does not require them.

### Closures

Closure are an important piece of the Groovy build scripts. Every block is a closure under the hood.

Because dealing with Groovy closure from Kotlin (and Java) is cumbersome, Gradle allows to use `Action<T>` instead. For all types instantiated by Gradle (tasks, extensions, [newInstance()](https://docs.gradle.org/current/kotlin-dsl/gradle/org.gradle.api.model/-object-factory/new-instance.html), etc..), the Gradle runtime decorates all functions with a single `Action<T>` parameter with an matching function accepting a closure ([doc](https://docs.gradle.org/current/userguide/kotlin_dsl.html#groovy_closures_from_kotlin)).

For an example, the Kotlin code below:

```kotlin
open class MyExtension {
  fun doStuff(action: Action<Spec>) {
    // ...
  }
} 
```

can be called from groovy with a closure:

```groovy
myExtension {
  doStuff {
    // This is a closure even though groovy doesn't know about Action  
    // ...  
  }
} 
```

## Difference with `build.gradle.kts` scripts

If you are used to writing `build.gradle.kts` files, you may use the `kotlin-dsl` Gradle plugin to write your plugins.

`kotlin-dsl` configures the Kotlin compiler so that you can use precompiled scripts plugins and/or write similar syntax in your regular `.kt` files.

The `kotlin-dls` plugin:
* applies `"java-gradle-plugin"`.
* applies `kotlin-embedded` to use the same Kotlin embeeded version as your Gradle distribution. 
* applies the `&#96;kotlin-dsl-precompiled-script-plugins&#96;` allowing to use `build.gradle.kts` files.
* adds `gradleKotlinDsl()` to the `compileOnly` configuration.
* configures the `sam-with-receiver` Kotlin compiler plugin to transform `it.` usages into `this.`.
* configures the `kotlin-assignment` Kotlin compiler plugin to allow setting `Property` with the `=` operator.
* sets Kotlin `apiVersion` and `languageVersion` according to Gradle [compatibility [atrix](https://docs.gradle.org/current/userguide/compatibility.html#kotlin). 
* adds the `-Xsam-conversions=class` compiler option.
* adds others compiler options for compatibility:
  ** `-java-parameters` to support https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Parameter.html[Java 8 Parameter] class and getting method parameters through reflection
  ** `-Xjvm-default=all` to add link:https://kotlinlang.org/docs/java-to-kotlin-interop.html#default-methods-in-interfaces[Default methods in interfaces]
  ** `-Xjsr305=strict` for https://kotlinlang.org/docs/java-interop.html#compiler-configuration[increased null safety]

This is a significant departure from the baseline Kotlin configuration so be aware of the tradeoffs when using `kotlin-dsl`. 

Also, `kotlin-dsl` targets the Kotlin embedded version of your current distribution. If you want to be compatible with lower versions of Gradle, using the `com.jetbrains.kotlin.jvm` plugin provides more flexibility.  