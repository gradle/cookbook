# Ensuring Configuration Cache compatibility

The configuration and execution phase of builds always take the longest.
You may already know about the build cache, which saves execution time by reusing outputs from past builds. 
The new [Configuration Cache](https://docs.gradle.org/current/userguide/configuration_cache.html) feature reuses configuration results from past builds to speed up the configuration phase.

If you keep an eye on [Gradle public roadmap updates](https://roadmap.gradle.org),
you've probably noticed lots of work on the configuration cache.
This feature is almost finished incubating -- expect to see a stable release soon.
Still, there is a lot of work to be done in the plugins,
many Gradle tasks still need to have configuration cache compatibility.
We track the status on [this GitHub Issue](https://github.com/gradle/gradle/issues/13490), and any contributions are more than welcome.

On this page, we focus on tips for plugins developers and contributors who want to fix Configuration Cache (CC) compatibility in their projects.

## Main Documentation

For Configuration Cache adoption and troubleshooting steps, you can already find information
in the main documentation:

- [Troubleshooting Configuration Cache compatibility](https://docs.gradle.org/current/userguide/configuration_cache.html#config_cache:troubleshooting)
- [Configuration Cache Adoption Steps](https://docs.gradle.org/current/userguide/configuration_cache.html#config_cache:adoption)
- [Auto-testing configuration Cache compatibility](https://docs.gradle.org/current/userguide/configuration_cache.html#config_cache:testing)


## FAQ

### How to fix "invocation of 'Task.project' at execution time is unsupported"?

In essence, you should use an alternative service that provides similar functionality. See "Using the Project object" in the [Configuration Cache documentation](https://docs.gradle.org/current/userguide/configuration_cache.html#config_cache:requirements:use_project_during_execution). See a sample [fix](https://github.com/gradle/gradle/pull/21555/files#r948986470).

Also, the configuration cache serialization logic handles `Provider<T>` specially and correctly serializes their values; use `Property<T>` and the providers returned from `org.gradle.api.provider.ProviderFactory` to store values captured from the project or the properties.

### How to fix "cannot serialize Gradle script object references"?

This problem most commonly signals that a member of the enclosing script scope, for instance, a member of the `Project` object in the case of project scripts, is being used by a task action:

```kotlin
tasks.register("printProjectVersion") {
    doLast {
        println(version) // <---- Project.version captured by a task action
    }
}
```

A common example is when the `configurations` property or a configuration object (like `runtimeClasspath`) is used by a doLast/doFirst action. Avoid accessing such objects during execution. See a [fix](https://github.com/gradle/gradle/pull/21555/files#r948983984).
