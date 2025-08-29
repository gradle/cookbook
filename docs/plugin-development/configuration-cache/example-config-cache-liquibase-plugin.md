---
title: Case Study - Enabling Gradle Configuration Cache Support in the Liquibase Plugin
description: >
    This page outlines the changes made to the Liquibase Gradle plugin to add support for Gradle's Configuration Cache.
---

## 1. Summary

This page outlines the changes made to the Liquibase Gradle plugin to add support for Gradle's **Configuration Cache**. The primary goal was to improve build performance by making the plugin's tasks compatible with this powerful Gradle feature.

The core of the work involved refactoring the `LiquibaseTask` and its helper classes to **decouple the task execution logic from the Gradle `Project` model**. This ensures that the task's configuration can be cached and reused across builds, leading to significantly faster development cycles.

---

## 2. The Problem: Configuration Cache Incompatibility

Gradle's Configuration Cache is a feature that dramatically speeds up builds by serializing the task graph and reusing it for subsequent builds. This allows Gradle to skip the expensive "configuration" phase entirely.

For the cache to work, a critical rule must be followed: **tasks must not access the `Project` object or related services during their execution phase**. The `Project` model is not serializable and is only available during configuration.

The previous implementation of the plugin violated this rule in several ways:

* **Direct Project Access in `LiquibaseTask`:** The main task action (`@TaskAction`) directly accessed the project's `liquibase` extension to get its configuration at execution time.

    ```groovy
    // Violation inside LiquibaseTask's execution logic:
    def activities = project.liquibase.activities
    def runList = project.liquibase.runList
    jvmArgs(project.liquibase.jvmArgs)
    ```

* **Project Dependency in `ArgumentBuilder`:** The `ArgumentBuilder` class, used during task execution, held a reference to the `project` and used it to access project properties, the logger, and the build directory.

    ```groovy
    // Violations inside ArgumentBuilder:
    project.properties.findAll { ... }
    commandArguments += "--output-directory=${project.buildDir}/database/docs"
    ```

* **Runtime Configuration Resolution:** The task resolved its `liquibaseRuntime` classpath inside its execution logic instead of during the configuration phase.

    ```groovy
    // Violation inside LiquibaseTask's execution logic:
    def classpath = project.configurations.getByName(LiquibasePlugin.LIQUIBASE_RUNTIME_CONFIGURATION)
    ```

These violations made the plugin incompatible with the configuration cache, forcing Gradle to reconfigure the project on every single run and preventing users from benefiting from the potential performance gains.

---

## 3. The Solution: Decoupling Execution from Configuration

The solution was to refactor the plugin to adopt a pattern where all necessary information is gathered from the `Project` during the **configuration phase** and passed into the task as **serializable inputs**. The task's execution logic then operates only on these inputs, with no knowledge of the `Project` object.

### Step 1: Introduce Serializable Data Transfer Objects (DTOs)

Two new classes were created to act as simple, serializable data containers:

1.  **`LiquibaseInfo`**: This class is responsible for carrying project-level information that the `ArgumentBuilder` needs.
* *See the new class file: [`LiquibaseInfo.groovy`]([lhttps://github.com/Nouran-11/liquibase-gradle-plugin/blob/fix-cc/src/main/groovy/org/liquibase/gradle/LiquibaseInfo.groovy$0])*

    ```groovy
    // org/liquibase/gradle/LiquibaseInfo.groovy
    class LiquibaseInfo {
        Logger logger
        File buildDir
        Map<String, Object> liquibaseProperties
        
        static LiquibaseInfo fromProject(Project project) {
            // ... logic to extract properties from the project ...
            return new LiquibaseInfo(project.logger, project.buildDir, liquibaseProperties)
        }
    }
    ```
1.  **`ProjectInfo`**: This class holds configuration from the `liquibase { ... }` extension block.
* *See the new class file: [`ProjectInfo.groovy`]([https://github.com/Nouran-11/liquibase-gradle-plugin/blob/fix-cc/src/main/groovy/org/liquibase/gradle/ProjectInfo.groovy$0])*
    ```groovy
    // org/liquibase/gradle/ProjectInfo.groovy
    class ProjectInfo {
        List<Activity> activities
        String runList
        List<String> jvmArgs

        static ProjectInfo fromProject(Project project) {
            // ... logic to extract data from project.liquibase extension ...
            return new ProjectInfo(activities, runList, jvmArgs)
        }
    }
    ```

The key feature of these classes is the static `fromProject()` factory method. This method acts as the bridge, safely extracting data at configuration time.

### Step 2: Refactor `ArgumentBuilder` to be Stateless

The `ArgumentBuilder` was refactored to remove its dependency on the `Project` object. Instead of holding a reference to the project, its methods now require a `LiquibaseInfo` object to be passed in.
* *See the full class changes: **[`ArgumentBuilder.groovy`]([https://github.com/liquibase/liquibase-gradle-plugin/blob/master/src/main/groovy/org/liquibase/gradle/ArgumentBuilder.groovy$0])** -> **[`ArgumentBuilder.groovy`]([https://github.com/Nouran-11/liquibase-gradle-plugin/blob/fix-cc/src/main/groovy/org/liquibase/gradle/ArgumentBuilder.groovy$0])***
**Before:**

```groovy
//  ArgumentBuilder.groovy
def buildLiquibaseArgs(Activity activity, commandName, supportedCommandArguments) {
    // ... code that used internal `project` reference ...
}
```

**After:**

```groovy
//  ArgumentBuilder.groovy
def buildLiquibaseArgs(Activity activity, commandName, supportedCommandArguments, LiquibaseInfo liquibaseInfo) {
    // ... code now uses liquibaseInfo.logger, liquibaseInfo.buildDir, etc. ...
}
```

## Step 3: Update LiquibaseTask to Use Inputs

The `@TaskAction` method (`exec` and its helpers) was rewritten to read from configured input properties instead of the project. This table highlights the specific transformations that eliminate configuration cache violations at execution time:
* *See the full class changes: **[`LiquibaseTask.groovy`]([https://github.com/liquibase/liquibase-gradle-plugin/blob/master/src/main/groovy/org/liquibase/gradle/LiquibaseTask.groovy$0])** -> **[`LiquibaseTask.groovy`]([https://github.com/Nouran-11/liquibase-gradle-plugin/blob/fix-cc/src/main/groovy/org/liquibase/gradle/LiquibaseTask.groovy$0])***

| Violation Type            | Before                                     | After                                        |
|---------------------------|-------------------------------------------|---------------------------------------------|
| Accessing Extension Data  | `jvmArgs(project.liquibase.jvmArgs)`      | `jvmArgs(projectInfo.get().jvmArgs)`       |
| Resolving Classpaths      | `setClasspath(project.configurations.getByName(...))` | The `classPath` property is assigned in the task's constructor                  |
| Accessing Project Properties | `argumentBuilder` reads `project.properties` | `argumentBuilder` receives `liquibaseInfo` with properties |
| Accessing buildDir        | `argumentBuilder` reads `project.buildDir` | `argumentBuilder` receives `liquibaseInfo` with build dir |
---

## 4. Verification: The Configuration Cache Test

To provide concrete proof of these improvements, a dedicated integration test was added. This test runs a build with the configuration cache enabled, verifying that the plugin's tasks execute successfully and are correctly stored and retrieved from the cache on subsequent runs. This confirms that the refactoring was successful and the plugin is now fully compatible.  

See the test in action: [Configuration Cache Test]([https://github.com/Nouran-11/liquibase-gradle-plugin/blob/master/src/test/groovy/org/liquibase/gradle/ConfigurationCacheSpec.groovy$0])

---

## 5. Conclusion

In conclusion, this refactoring effort was centered on achieving full compatibility with Gradle's Configuration Cache. By decoupling task execution from the project model, the plugin now allows Gradle to serialize its state and entirely skip the configuration phase on subsequent runs. This unlocks the core performance promise of the configuration cache, resulting in a dramatically faster and more efficient development workflow.

