---
title: "Publishing to Maven Central"
description: >
  Maven Central”, is one of the most popular repositories for publishing artifacts in open source projects.
  You can publish your Gradle projects, too.
---

# Publishing to Maven Central

The [Maven Central Repository](https://central.sonatype.com/) service by Sonatype, known in the community as “Maven Central”, is one of the most popular repositories for publishing open source libraries and other distributable artifacts in the Java and Kotlin ecosystems.

Many developers who use Gradle publish their open source projects to Maven Central. The publishing flow for Maven Central differs significantly from publishing to a generic Maven repository using the [Maven Publish Plugin](https://docs.gradle.org/current/userguide/publishing_maven.html). It involves additional requirements for the artifacts themselves (such as metadata files and signed JARs), as well as for the publishing process (including a staging workflow and server-side verification). Hence, this solution page provides some tips and references.

## Plugins for publishing to Maven Central

As of now, there is no officially recommended plugin for publishing to Maven Central. However, a variety of community-supported plugins and tools are [available](https://plugins.gradle.org/search?term=Maven+Central) on the Gradle Plugin Portal. Below are some of the most popular options plugins, ranked by GitHub stars and having more than 10 stars:

* [vanniktech/gradle-maven-publish-plugin](https://github.com/vanniktech/gradle-maven-publish-plugin)   
* [JReleaser](https://jreleaser.org/)   
* [GradleUp/nmcp](https://github.com/GradleUp/nmcp)   
* [deepmedia/MavenDeployer](https://github.com/deepmedia/MavenDeployer)   
* [DanySK/maven-central-gradle-plugin](https://plugins.gradle.org/plugin/org.danilopianini.maven-central-gradle-plugin)   
* [yananhub/flying-gradle-plugin](https://github.com/yananhub/flying-gradle-plugin) 

Note that not all listed plugins are actively maintained, and one has to choose carefully based on their use cases. See the recommendations below.

## Breaking Changes in Maven Central on June 2025

As [announced](https://central.sonatype.org/news/20250326_ossrh_sunset/) by Sonatype, the [Sonatype OSS Repository Hosting service (OSSRH)](https://central.sonatype.org/publish/publish-guide/) will reach end-of-life on June 30th, 2025\. OSSRH is an old and deprecated component of the [Maven Central Repository](https://central.sonatype.com/) service. 

With the sunset of OSSRH, Maven Central is transitioning to a new hosting implementation, which includes a complete redesign of its APIs. As a result, Maven Central publishing plugins must be updated to remain compatible once the OSSRH compatibility layer is removed in June 2025\.

We invite the plugin maintainers to update their plugins and add support for the new APIs. 

### Compatible Plugins

Several plugins already report compatibility with the new APIs, and new plugins have been developed specifically for the updated implementation. Among the plugins listed above, the following are known to be mostly compatible based on their changelogs:

* [vanniktech/gradle-maven-publish-plugin](https://github.com/vanniktech/gradle-maven-publish-plugin) (common and base versions)  
* [JReleaser](https://jreleaser.org/)   
* [GradleUp/nmcp](https://github.com/GradleUp/nmcp) 

## Choosing a plugin for Maven Central publishing

Choosing the right plugin might be non-trivial, as they have different feature sets and might include different developer experiences in your particular publishing use case. For common Java and Kotlin projects that do not include custom artifacts and distributions, the following selection algorithm can be considered:

1. If you want to automate the whole release process (packaging, signing, changelog generation, etc.) in a single call and are ready to follow conventions, consider using [JReleaser](https://jreleaser.org/).  
2. If you want only a publishing step that follows Maven Central conventions, use [vanniktech/gradle-maven-publish-plugin](https://github.com/vanniktech/gradle-maven-publish-plugin) in the default configuration. It can publish your Java, Android, and Kotlin libraries, including sources and documentation, to Maven Central or any other Nexus instance.  
3. If you want to customize the whole publishing flow, e.g., for staging or custom artifact types, use [GradleUp/nmcp](https://github.com/GradleUp/nmcp) or [com.vanniktech.maven.publish.base](https://vanniktech.github.io/gradle-maven-publish-plugin/base/). 

\> DISCLAIMER: This algorithm does not represent an official endorsement by Gradle, Inc. Your mileage may vary, and there are other plugins out there.

## References

* [Deploying to OSSRH / Maven Central with Gradle](https://central.sonatype.org/publish/publish-gradle/) \- official documentation page on the Maven Central Documentation site  
* [GSoC 2025 \- Maven Central publishing for Gradle with new APIs](https://community.gradle.org/events/gsoc/2025/maven-central-publishing-with-new-api/) \- Google Summer of Code 2025 project targeting the problem area

## Discuss

To support this effort, we have created a new **\#maven-central-publishing** channel on the [Gradle Community Slack](https://slack.gradle.org/).
