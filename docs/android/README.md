# Gradle for Android

Gradle is one of the most popular build tools in the Android ecosystem. It is used for [Kotlin and Kotlin Multiplatform](../kotlin/README.md), [Flutter](https://flutter.dev/), [React Native](https://reactnative.dev/), and other toolchains.

## Official Documentation

The official documentation for Android development with Gradle is provided by Google and the community on the Android Developers site. You can find this documentation [here](https://developer.android.com/build).

## Featured Recipes

### **1. Gradle Build Overview**
Learn how Gradle builds Android applications using the **Android Gradle Plugin (AGP)** and explore key concepts, such as build variants and build types like `debug` and `release`. This section will help you understand the structure of a standard Android project.

#### Key Concepts:
- **Build Variants**: Configure different variants of your application (e.g., `debug`, `release`).
- **Build Types**: Use predefined types or define custom types for different stages of development.

```groovy
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

For more details, visit the [Gradle build overview](https://developer.android.com/build/gradle-build-overview).


### **2. Android Build Management**
Configure your Android builds for better packaging to test, build, sign, and distribute your app bundles and APKs. Learn to configure **build flavors**, set up **product dimensions**, and create **dynamic delivery modules** for efficient app distribution.

#### Example of Build Flavors:

```groovy
android {
    flavorDimensions "version"
    productFlavors {
        free {
            dimension "version"
        }
        paid {
            dimension "version"
        }
    }
}
```

For more insights, visit [Android build management](https://developer.android.com/build).


### **3. Managing Dependencies**
Gradle allows you to add external binaries or library modules as dependencies. Proper dependency management is essential for maintaining large Android projects. You can now resolve version conflicts using **version catalogs** and handle dependencies for **Kotlin Multiplatform** projects.

#### Managing Dependencies in `build.gradle`:
```groovy
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation platform('com.google.firebase:firebase-bom:26.7.0')
}
```

Visit the [Managing dependencies](https://developer.android.com/build/dependencies) page for more.


### **4. Building Projects**
Use the **Gradle Wrapper** to execute tasks for building and deploying APKs or app bundles. Learn to automate build pipelines with **CI/CD** systems, providing seamless integration with cloud platforms.

#### Example of building an APK:
```bash
./gradlew assembleDebug
```

This command compiles and packages the APK in `debug` mode. Learn more about building Android projects at [Building project](https://developer.android.com/build/building-cmdline).


### **5. Optimizing Builds**
You can reduce build times by configuring **build caching**, **parallel execution**, and **Gradle Daemon** tuning, which is crucial for large-scale projects. These techniques optimize performance during the development cycle.

#### Example of enabling parallel execution:
```groovy
org.gradle.parallel=true
```

For further optimizations, visit [Optimizing Builds](https://developer.android.com/build/optimize-your-build).


### **6. Extending AGP**
Extend the **Android Gradle Plugin (AGP)** by creating custom plugins or tasks that integrate with existing build tasks. This allows you to tailor your build process to fit your project’s unique requirements.

#### Example of a custom task in Groovy:
```groovy
task hello {
    doLast {
        println 'Hello, Android Developers!'
    }
}
```

For details on how to extend AGP, check out [Extending AGP](https://developer.android.com/build/extend-agp).



### **7. Migration Guide**
If you are migrating from **Groovy** to **Kotlin DSL**, follow the comprehensive steps to ensure a smooth transition. This guide covers moving from **Kapt** to **KSP** for annotation processing.

#### Example of Kotlin DSL in `build.gradle.kts`:
```kotlin
plugins {
    id("com.android.application")
    kotlin("android")
}
```

For a complete migration guide, visit [Migration Guide](https://developer.android.com/build/migrate-to-kotlin-dsl).


### **8. Troubleshooting AGP**
This guide provides solutions to common issues with the **Android Gradle Plugin (AGP)**, including resolving dependency conflicts, handling build script errors, and identifying performance bottlenecks.

#### Common Fix: Handling Dependency Resolution Failures
```groovy
configurations.all {
    resolutionStrategy {
        force 'com.google.guava:guava:27.0.1-android'
    }
}
```

For more troubleshooting tips, visit [Troubleshooting AGP](https://developer.android.com/build/troubleshoot).


## References

- [Official Documentation](https://developer.android.com/build)
- [AGP releases](https://developer.android.com/build/releases/gradle-plugin)
- [Android Gradle Plugin API reference](https://developer.android.com/reference/tools/gradle-api)
