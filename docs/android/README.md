
# Gradle for Android

Gradle is one of the most popular build tools in the Android ecosystem. It is used for [Kotlin and Kotlin Multiplatform](../kotlin/README.md), [Flutter](https://flutter.dev/), [React Native](https://reactnative.dev/), and other toolchains.

## Official Documentation

The official documentation for Android development with Gradle is provided by Google and the community on the Android Developers site. You can find this documentation [here](https://developer.android.com/build).

## Featured Recipes

- [Gradle build overview](https://developer.android.com/build/gradle-build-overview) - Understand how Gradle builds Android applications with Gradle using Android Gradle Plugin (AGP), key concepts, and the structure of a standard Android project.
  
- [Android build management](https://developer.android.com/build) - Configure your Android builds for better packaging to test, build, sign, and distribute your app bundles and APKs.
  
- [Managing dependencies](https://developer.android.com/build/dependencies) - Configure remote repositories to add external binaries or other library modules into your build as dependencies.
  
- [Building project](https://developer.android.com/build/building-cmdline) - Execute tasks available for Android projects using the Gradle wrapper to build and deploy APKs and app bundles.
  
- [Optimizing Builds](https://developer.android.com/build/optimize-your-build) - Configure your builds to decrease the build time, resulting in faster development of your Android projects.
  
- [Extending AGP](https://developer.android.com/build/extend-agp) - AGP contains extension points for plugins to control build inputs and extend its functionality through new steps that can be integrated with standard build tasks.
  
- [Migration Guide](https://developer.android.com/build/migrate-to-kotlin-dsl) - Guide for migrating from Groovy to Kotlin DSL, version catalogs, Kapt (Kotlin Annotation Processing Tool) to KSP (Kotlin Symbol Processing).
  
- [Troubleshooting AGP](https://developer.android.com/build/troubleshoot) - Troubleshooting guide if you encounter issues with the Android Gradle Plugin (AGP).

## Expanded Sections

### **1. Gradle Build Overview**
Gradle builds Android applications using the **Android Gradle Plugin (AGP)**, which provides the flexibility to define build types, product flavors, and dependencies. Understanding these key concepts is crucial for creating efficient and scalable Android projects.

#### Key Concepts:
- **Build Variants**: Manage multiple variants of your app such as `debug` and `release`.
- **Build Types**: Use predefined types (e.g., `debug`, `release`) to control various build configurations.
- **Kotlin DSL**: Gradle uses Kotlin DSL by default for better type safety and editor support in Android builds.

#### Example in Kotlin DSL:
```kotlin
android {
    buildTypes {
        release {
            minifyEnabled(true)
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```
For more details, visit the [Gradle build overview](https://developer.android.com/build/gradle-build-overview).

### **2. Android Build Management**
Android build management allows you to define various configurations for packaging, testing, signing, and distributing your app. The ability to manage product flavors, build types, and dynamic delivery modules ensures flexibility across different versions of your app.

#### Example of Build Flavors:
```kotlin
android {
    flavorDimensions("version")
    productFlavors {
        free {
            dimension = "version"
        }
        paid {
            dimension = "version"
        }
    }
}
```
For more information, visit [Android build management](https://developer.android.com/build).

### **3. Managing Dependencies**
Gradle enables the addition of external binaries or library modules as dependencies, which is essential for maintaining large Android projects. You can also use **version catalogs** to resolve dependency version conflicts and handle dependencies in **Kotlin Multiplatform** projects.

#### Example of Dependencies in Kotlin DSL:
```kotlin
dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation(platform("com.google.firebase:firebase-bom:26.7.0"))
}
```
For more information, visit [Managing dependencies](https://developer.android.com/build/dependencies).

### **4. Building Projects**

Gradle provides tasks for building your Android projects, including compiling code, packaging APKs, and generating app bundles. Using the **Gradle wrapper**, you can ensure build consistency across environments.

#### Example of Building an APK with Kotlin DSL:

```kotlin
tasks.register("customAssembleDebug") {
    doLast {
        println("Building APK in debug mode...")
    }
}
```
This Kotlin example registers a task to build the APK in debug mode. For more detailed steps on building projects and packaging apps, visit the [Building Projects Guide](https://developer.android.com/build/building-cmdline).

### **5. Optimizing Builds**
Gradle provides various options to improve build performance, including **build caching**, **parallel execution**, and **Gradle Daemon tuning**. These methods are especially beneficial for larger projects, helping to reduce build times and enhance productivity.

#### Example of Enabling Parallel Execution:
```properties
org.gradle.parallel=true
```

To explore more optimization techniques and configure your builds for better performance, check out our detailed guide: [Gradle Build Optimization](./optimization.md). 

For further information, visit the **official**  [Optimizing Builds Guide](https://developer.android.com/build/optimize-your-build).

### **6. Extending AGP**
You can extend the Android Gradle Plugin (AGP) by writing custom tasks and plugins to integrate new functionality into the existing build system. This allows for advanced customization and flexibility in your build processes.

#### Example of a Custom Task:
```kotlin
tasks.register("printHelloMessage") {
    doLast {
        println("Hello, Android Developers!")
    }
}
```
For more information, visit [Extending AGP](https://developer.android.com/build/extend-agp).

### **7. Migration Guide**

With the shift from **Groovy** to **Kotlin DSL** in the Android ecosystem, migrating your build scripts is essential. This guide helps you transition your build configurations and annotation processing from **Kapt** to **KSP**.

For detailed steps and best practices, refer to the [official Migration Guide](https://developer.android.com/build/migrate-to-kotlin-dsl).

### **8. Troubleshooting AGP**

Encountering issues with the Android Gradle Plugin (AGP) is common, and this section provides solutions to common problems such as dependency resolution failures, build script errors, and performance bottlenecks.

#### Example of Dependency Conflict Resolution:
```kotlin
configurations.all {
    resolutionStrategy {
        force("com.google.guava:guava:27.0.1-android")
    }
}
```
For more troubleshooting tips, visit the **official**  [Troubleshooting AGP](https://developer.android.com/build/troubleshoot) page.

You can also check the [Gradle Troubleshooting](./troubleshooting.md) page for additional help with Gradle-specific issues.

## References

- [Official Documentation](https://developer.android.com/build)
- [AGP releases](https://developer.android.com/build/releases/gradle-plugin)
- [Android Gradle Plugin API reference](https://developer.android.com/reference/tools/gradle-api)
