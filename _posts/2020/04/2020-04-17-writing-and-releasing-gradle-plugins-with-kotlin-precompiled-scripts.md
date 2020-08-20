---
date: 2020-04-17
title: "How to write and publish Gradle precompiled script plugins"
figure: /assets/images/posts/2020/04/writing-and-releasing-gradle-plugins-with-kotlin-precompiled-scripts/gradle-kotlin.png
figcaption: © Chirag Kunder
figalt: Gradle and Kotlin
description: In this article I will walk you through on how to create a precompiled script plugin and publish it on the Gradle Plugins portal.
keywords:
- kotlin

categories: guide
comments: true
---

[Gradle] has often been portrayed as an imperative build automation tool, in contrast to Maven, which is of course
declarative as it relies on XML. However, Gradle has been rapidly improving and offering more and more opportunities to
write build scripts more expressively and declaratively. One of such improvements are *[precompiled script plugins]*,
which allow creating Gradle plugins in the same exact way as you write your project's build script. In this article
I will walk you through on how to create a precompiled script plugin and publish it on the [Gradle Plugins portal]. 
For the sake of an example I will use my [gradle-kotlin-setup-plugin], which you can use as reference.

<!--more-->

First of all, we will need to create a project directory and add some files to it.
```
gradle-kotlin-setup-plugin - project dir
    src/main/kotlin
        com/driver733
            gradle-kotlin-setup-plugin.gradle.kts 
                - precompiled script plugin
    build.gradle.kts - project build script
```

Next, we need to add the `kotlin-dsl` plugin (to get Gradle Kotlin DSL support), and the 
`com.gradle.plugin-publish` plugin (which will help us to publish our plugin to the Gradle Plugins portal) to our project's
build script.

```kotlin
// build.gradle.kts

plugins {
    `kotlin-dsl`
    id("com.gradle.plugin-publish") version "0.11.0"
}

group = "com.driver733"

repositories {
    mavenCentral()
    gradlePluginPortal()
}
```

Next, we need to write our precompiled script plugin. In other words, we need to write a build script, similar to the one
we have in our project root, that will be implicitly transformed into a gradle plugin. 

Let's say that we want our plugin to apply a `kotlin-gradle-plugin` plugin. To do this, we first need to add a corresponding
dependency to our project's build script.

```kotlin
// build.gradle.kts

dependencies {
    implementation(kotlin("gradle-plugin", "1.3.72"))
}
```
Then we can add the `kotlin-gradle-plugin` to our precompiled script plugin. Again, this done just as if you were adding
it your project's build script. The only difference is that instead of specifying the plugin's version in the `plugins`
block, you instead specify it in the dependency statement in your project's build script. 

```kotlin
// src/main/kotlin/com.driver733/gradle-kotlin-setup-plugin.gradle.kts

plugins {
    kotlin("jvm")
}
```

Now, let's say that we also want our plugin to include some dependencies that are common for Kotlin projects.
This is done similar to how you would add a dependency to your project using the project's build script.

```kotlin
// src/main/kotlin/com/driver733/gradle-kotlin-setup-plugin.gradle.kts

dependencies {
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.3.72")

    testImplementation("org.jetbrains.kotlin:kotlin-test:1.3.72")
}
```

Alright, seems like our plugin is ready! To use it, you just have to add it to the plugins block of your project's
build script. (Notice that the plugin's id matches the filename of the precompiled script plugin)

```kotlin
// src/main/kotlin/com/driver733/gradle-kotlin-setup-plugin.gradle.kts

plugins {
  id("gradle-kotlin-setup-plugin") version "X.X"
}
```

However, before using our plugin in other projects, we first need to release it to a repository. I will demonstrate how
to publish your plugin to the Gradle Plugins portal, but you can also use your plugin locally if you publish it
to your local maven repository using the `maven-publish` plugin. (Which allows you to test your plugin before making it public)

All plugins that are published to the Gradle Plugins portal are required to have some informational metadata.
Copy this block into your project's build script, replacing field values with your plugin's metadata.

```kotlin
// build.gradle.kts

pluginBundle {
    website = "https://github.com/driver733/gradle-kotlin-setup-plugin"
    vcsUrl = "https://github.com/driver733/gradle-kotlin-setup-plugin.git"
    tags = listOf("kotlin", "setup")
}

gradlePlugin
    .plugins
    // replace the right part of the comparison 
    // with your package and precompiled script filename
    .find { it.name == "com.driver733.gradle-kotlin-setup-plugin" }!!
    .apply {
        id = "com.driver733.gradle-kotlin-setup-plugin"
        displayName = "A plugin that sets up kotlin in your project"
        description = "A plugin that sets up kotlin dependencies, plugins and build settings"
    }
```
 
Finally, you need to acquire a `key` and a `secret`, by [creating an account] on the Gradle Plugins portal. 
Once you do that, all you have to do to publish your plugin is to run the `publishPlugins` task of 
the `com.gradle.plugin-publish` plugin.

```
gradle publishPlugins -Pgradle.publish.key=<key> -Pgradle.publish.secret=<secret>
```


[Gradle]: https://en.wikipedia.org/wiki/Gradle
[precompiled script plugins]: https://docs.gradle.org/current/userguide/kotlin_dsl.html#kotdsl:precompiled_plugins
[Gradle Plugins portal]: https://plugins.gradle.org
[gradle-kotlin-setup-plugin]: https://github.com/driver733/gradle-kotlin-setup-plugin
[creating an account]: https://guides.gradle.org/publishing-plugins-to-gradle-plugin-portal/

