---
date: 2020-07-16
title: "Testing Gradle precompiled script plugins"
figure: /assets/images/posts/2020/08/how-to-write-tests-for-gradle-precompiled-script-plugins/gradle-kotlin.png
figcaption: © JetBrains Blog
figalt: Gradle and Kotlin
description: A guild for writing integration tests for Gradle precompiled script plugins written in Kotlin.
keywords:
- kotlin
- gradle

categories: guide
comments: true
---

In the [previous article] I discussed how to write and publish [Gradle] [precompiled script plugins]. Today, I want to
show you how to write integration tests for them.

<!--more-->

As before I will be using my [gradle-kotlin-setup-plugin] Gradle plugin, which you can use as a reference.

Let's start be setting up the source directory for our tests.
```
gradle-kotlin-setup-plugin
    src/test/kotlin
        com/driver733
            GradleKotlinSetupPluginTest.kts
```

Next, we'll need to set up a test class and a test method. I am using the [KoTest] testing framework, but any other framework
will work fine too.

```kotlin
class GradleKotlinSetupPluginTest : FunSpec() {
    init {
        test("build") {
            //
        }
    }   
}
```

An integration test for a Gradle plugin consist of the following parts:

1. Create a temporary Gradle project folder with
the `setting.gradle.kts` and `build.gradle.kts` files in it, which describe a project which uses your plugin.
2. Run a Gradle command using the [GradleRunner], specifying the project directory from the first step.
3. Verify the Gradle command result, output and anything else you expect to happen.

Let's go through the first step:

We start by creating a temporary directory by using the `createTempDir()` standard library method.
Then we put the `settings.gradle.kts` file inside this directory, which declares out project.
Lastly, we add the `build.gradle.kts` file and add our plugin to the [Plugins DSL] block.
If your plugin relies on third-party dependencies, you should also add a repository, which provides these dependencies.

Here's the code for the first step of the test:
```kotlin
val projectDir = createTempDir().apply {
        resolve("setting.gradle.kts").apply {
            appendText("rootProject.name = \"gradle-kotlin-setup-plugin-test\"")
        }
        resolve("build.gradle.kts").apply {
            appendText(
                """
                    plugins {
                        id("com.driver733.gradle-kotlin-setup-plugin")
                    }
                    repositories {
                        mavenCentral()
                    }
            """
            )
        }
    }
```

The [GradleRunner] allows to run a Gradle build using just a few lines of code:

```kotlin
val buildResult = GradleRunner.create()
    .withProjectDir(projectDir)
    .withArguments("build")
    .withPluginClasspath()
    .build()
```

The build result allows us to check the outcome of each task that was executed
```kotlin
buildResult.task("build")?.outcome shouldBe TaskOutcome.SUCCESS        
```
as well as the output of the build:
```kotlin
output.contains("BUILD SUCCESSFUL")
```

Below is the resulting code for our test:
```kotlin
class GradleKotlinSetupPluginTest : FunSpec() {
    val projectDir = createTempDir().apply {
        resolve("setting.gradle.kts").apply {
            appendText("rootProject.name = \"gradle-kotlin-setup-plugin-test\"")
        }
        resolve("build.gradle.kts").apply {
            appendText(
                """
                    plugins {
                        id("com.driver733.gradle-kotlin-setup-plugin")
                    }
                    repositories {
                        mavenCentral()
                    }
            """
            )
        }
    }

    init {
        test("build") {
            val actual = GradleRunner.create()
                .withProjectDir(projectDir)
                .withArguments("build")
                .withPluginClasspath()
                .build()
            
            actual.output shouldContainIgnoringCase "kotlin"
            actual.task(":build")?.outcome shouldBe TaskOutcome.SUCCESS
            output.contains("BUILD SUCCESSFUL")
        }
    }   
}
```

In case you need more examples, feel free to use the tests for my [gradle-kotlin-setup-plugin] Gradle plugin. 


[previous article]: /2020/04/17/writing-and-releasing-gradle-plugins-with-kotlin-precompiled-scripts.html
[KoTest]: https://github.com/kotest/kotest
[Gradle]: https://en.wikipedia.org/wiki/Gradle
[precompiled script plugins]: https://docs.gradle.org/current/userguide/kotlin_dsl.html#kotdsl:precompiled_plugins
[GradleRunner]: https://docs.gradle.org/current/javadoc/org/gradle/testkit/runner/GradleRunner.html
[Plugins DSL]: https://docs.gradle.org/current/dsl/org.gradle.plugin.use.PluginDependenciesSpec.html
[gradle-kotlin-setup-plugin]: https://github.com/driver733/gradle-kotlin-setup-plugin

