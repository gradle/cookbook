# Troubleshooting Android Gradle Plugin (AGP)

This page serves as a resource for resolving common issues encountered while working with the Android Gradle Plugin (AGP). Below are some typical problems and solutions.

## Common Issues

### 1. Dependency Resolution Failures
Dependency conflicts can arise when multiple libraries require different versions of the same dependency. Use the following example to enforce a specific version:

```kotlin
configurations.all {
    resolutionStrategy {
        force("com.google.guava:guava:27.0.1-android")
    }
}
```

### 2. Build Script Errors
Errors in the build script can prevent the project from compiling. Ensure that all plugins and dependencies are correctly defined. Review the logs for specific error messages that can guide you to the problem.

### 3. Performance Bottlenecks
Build performance issues may occur due to improper configurations. To optimize your build time, consider using techniques like enabling build caching and parallel execution.

## Additional Resources
- [Troubleshooting AGP](https://developer.android.com/build/troubleshoot) page.

## Contributing
Feel free to add additional tips or common issues you encounter. Contributions are welcome to improve this troubleshooting guide.
