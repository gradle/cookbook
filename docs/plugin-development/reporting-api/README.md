---
title: Gradle Reporting API
description: >
    Gradle has a reporting system that tasks creating reports can expose,
    letting users configure formats and output locations. 
---

Gradle has a reporting system that tasks creating reports can expose, letting users configure formats and output locations. 
For plugin developers that would like closer integration with Gradle systems, to integrate the [Reporting API](https://docs.gradle.org/current/javadoc/org/gradle/api/reporting/Reporting.html) 
into your own Gradle plugin, two things are needed:
a [ReportContainer](https://docs.gradle.org/current/javadoc/org/gradle/api/reporting/ReportContainer.html) class and
a [Report](https://docs.gradle.org/current/javadoc/org/gradle/api/reporting/Report.html) 
object.

We’ll be working with the [VerifyPluginProjectConfigurationTask](https://github.com/JetBrains/intellij-platform-gradle-plugin/blob/476130fa41347ef4e5480fb44b9d454e51aa7a18/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/VerifyPluginProjectConfigurationTask.kt#L44), a task from the [IntelliJ Platform Gradle Plugin](https://github.com/JetBrains/intellij-platform-gradle-plugin/tree/476130fa41347ef4e5480fb44b9d454e51aa7a18) 
that does the simple job of validating a plugin project’s configuration.

## Declaring Reporting Support in a Task

Tasks that produce reports must implement the `Reporting` interface with a type argument that extends `ReportContainer`:

```kotlin
abstract class VerifyPluginProjectConfigurationTask : DefaultTask(), Reporting<VerifyPluginConfigurationReports> {
    // skipped lines
}
```

## Defining the `ReportContainer` Contract

`VerifyPluginConfigurationReports` is an interface that extends `ReportContainer<SingleFileReport>`. `SingleFileReport` 
is a useful wrapper around `Report` that represents that our output will be a single file. If your report was a group of files 
you’d want to put in a directory, of which a `DirectoryReport` would be used instead.

```kotlin
interface VerifyPluginConfigurationReports: ReportContainer<SingleFileReport> {
	@get:Nested
	val txt: SingleFileReport
}
```

This interface defines the public contract of reports available to users. Having multiple report formats, for example, text, 
html and/or markdown, can be defined like so:

```kotlin
interface VerifyPluginConfigurationReports: ReportContainer<SingleFileReport> {
	@get:Nested
	val txt: SingleFileReport

	@get:Nested
	val html: SingleFileReport

	@get:Nested
	val markdown: SingleFileReport		
}
```

If you need a custom report with additional properties beyond say `SingleFileReport`, you can define your own report type 
(e.g., an interface extending `SingleFileReport`) and still provide it from your `ReportContainer`.

Next, we need an implementation which will define how exactly our reports will be created:

```kotlin
class VerifyPluginConfigurationReportsImpl : VerifyPluginConfigurationReports {

}
```

## Implementing the `ReportContainer`

Providing an implementation of our report container requires that we override over 40 different methods. 
To avoid doing this, we can delegate access to these methods to another `ReportContainer` class with the help of `DelegatingReportContainer`. 

Since `DelegatingReportContainer`, `DefaultReportContainer` and `DefaultSingleFileReport` are internal classes, 
keep their usages inside your implementation so your plugin’s API only exposes public Gradle types like `ReportContainer` and `SingleFileReport`.

```kotlin
class VerifyPluginConfigurationReportsImpl @Inject constructor(
    owner: Describable, 
    objectFactory: ObjectFactory,
) : DelegatingReportContainer<SingleFileReport>(
        DefaultReportContainer.create(objectFactory,SingleFileReport::class.java) { factory -> 
            listOf(factory.instantiateReport(DefaultSingleFileReport::class.java, "txt", owner)) 
        }
     ), 
    VerifyPluginConfigurationReports {

}
```

`DefaultSingleFileReport` requires a name and a `Describable` object. 
And the `DefaultReportContainer` contains a `create` function that allows us to create a new report container by 
supplying an `ObjectFactory` instance, the type of reports that this container holds and a generator that creates those reports using `ReportGenerator`:

```kotlin
public interface ReportGenerator<T extends Report> { 
    Collection<T> generateReports(ReportFactory<T> factory);
}
```

Here, `generateReports` demands that we return a collection of our report types, so using the `ReportFactory` we can instantiate our report by 
passing in the required constructor arguments: name of the report and a `Describable` object.

Next, we must provide an implementation for our `txt` property by referencing the name of our report:

```kotlin
class VerifyPluginConfigurationReportsImpl @Inject constructor(
    owner: Describable,
    objectFactory: ObjectFactory,
) : DelegatingReportContainer<SingleFileReport>(
    DefaultReportContainer.create(objectFactory,SingleFileReport::class.java) { factory ->
        listOf(factory.instantiateReport(DefaultSingleFileReport::class.java, "txt", owner))
    }
),
    VerifyPluginConfigurationReports {
        
    override val txt: SingleFileReport get() = getByName("txt")

}
```

## Wiring the `ReportContainer` into the Task

After defining our report container and report objects, back to our task to start working with them:

```kotlin
abstract class VerifyPluginProjectConfigurationTask : DefaultTask(), Reporting<VerifyPluginConfigurationReports> { 
    // skipped lines
    
    @get:Inject 
    abstract val objectFactory: ObjectFactory 
    
    private val verifyPluginProjectReports : VerifyPluginConfigurationReports = objectFactory.newInstance(
        VerifyPluginConfigurationReportsImpl::class.java, 
        Describables.quoted("Task", identityPath)
    ) 
    // skipped lines
}
```

We will not be able to create an instance of `VerifyPluginConfigurationReportsImpl` because it’s a `final` class, so let’s  make it `open`:

```kotlin
open class VerifyPluginConfigurationReportsImpl @Inject constructor(
    owner: Describable, 
    objectFactory: ObjectFactory,
) /*skipped lines*/ {
    // skipped lines
}
```

Override these functions to rely on our reports property `verifyPluginProjectReports`:

```kotlin
abstract class VerifyPluginProjectConfigurationTask : DefaultTask(), Reporting<VerifyPluginConfigurationReports> {
	// skipped lines
    
    @Nested 
    override fun getReports(): VerifyPluginConfigurationReports = verifyPluginProjectReports
    
    override fun reports(closure: Closure<*>): VerifyPluginConfigurationReports { 
        return reports(ClosureBackedAction(closure)) 
    }
    
    override fun reports(configureAction: Action<in VerifyPluginConfigurationReports>): VerifyPluginConfigurationReports { 
        configureAction.execute(verifyPluginProjectReports)
        return verifyPluginProjectReports 
    } 
    // skipped lines
}
```

We annotate `getReports()` with `@Nested` so Gradle inspects the returned report container’s properties (like `txt`, `html`, `markdown`) and their annotations.

## Setting up Report Outputs

When configuring our task, we can go ahead to use our function to set up our reports:

```kotlin
reports { 
    txt.required.set(true)
    txt.outputLocation.convention(project.layout.buildDirectory.file("reports/verifyPluginConfiguration/report.txt"))
}
```

## Writing Data to Reports

Finally, writing to our reports can be done in this fashion:

```kotlin
if (verifyPluginProjectReports.txt.required.get()) { 
    verifyPluginProjectReports.txt.outputLocation.get().asFile.writeText(it)
}
```

## References

- https://docs.gradle.org/current/dsl/org.gradle.api.reporting.ReportContainer.html
- https://docs.gradle.org/current/dsl/org.gradle.api.reporting.Reporting.html
- https://docs.gradle.org/current/dsl/org.gradle.api.reporting.Report.html
- https://docs.gradle.org/current/dsl/org.gradle.api.reporting.SingleFileReport.html
- https://docs.gradle.org/current/dsl/org.gradle.api.reporting.DirectoryReport.html
- https://github.com/gradle/gradle/issues/7063

Tested with: Gradle 9.0.0