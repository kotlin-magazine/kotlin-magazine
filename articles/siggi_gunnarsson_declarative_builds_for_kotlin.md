# Declarative Gradle? Amper? What’s going on?!

We Kotlin developers have it pretty easy as it comes to configuring and executing our builds. We usually work with nice IDEs that help setting projects up for us, and a robust build system in Gradle. But there are some issues with Gradle. New projects are attempting to remedy these issues by changing how we work.

Gradle gives us a nice way to import dependencies, compile and package the project. It even helps with running tests. Not all languages have such a robust and "standardized" system. In Python, dependencies are declared by writing them into a `requirements.txt` file. Then you use `pip` to install everything into your (hopefully virtual) Python environment and pray that there are no dependency conflicts. Since there will be conflicts anyway, you can write explicit library versions into this `.txt` file based on the complaints from `pip` for a while before deciding maybe AI development isn't for you.

Back to Kotlin! For a simple project it's pretty easy to read the Groovy based `build.gradle` build files. Since you can now write `build.gradle.kts` in Kotlin, build files are also easy to write!

> #### Kotlin application `build.gradle.kts`

``` kotlin
plugins {
  application
}

repositories {
  mavenCentral()
}

dependencies {
  implementation(libs.someDependency)
}

tasks.register("funTask") {
  moveFiles()
}

tasks.getByName("build").dependsOn("funTask")

application {
  mainClass.set("MainClassFile")
}

```

This all looks straightforward. The project type is application. We declare some dependencies, and where they should be fetched from.

It’s nice to write build configurations in our favorite programming language, but that comes at a cost. Gradle is not able to statically analyze our build files when they contain arbitrary build logic. This limits how much our IDEs can help us, and can lead to hard-to-debug build issues.

We can mitigate this by following best practices:
- Only put declarative software definitions into build scripts
- Put all your custom build logic into Gradle plugins

Unfortunately, there’s nothing ensuring we follow best practices. Many projects pepper gnarly build logic around their build scripts. It's also not always obvious if your "declarative" build code really is just that. Was your new Gradle code "software definition" or did you actually write "build logic"? Did you spot the build logic cleverly hidden in the example?

## Enter: Declarative Gradle

The good folks at Gradle are aware of this, and have started work on a new kind of Gradle script file that only supports build configuration functionality. This should ensure that build logic is safely tucked away in build plugins. Although this work is still in a prototyping phase, let's look at an example of a simple Kotlin Multiplatform application. Note the “.dcl” file extension.

>### Kotlin application `build.gradle.dcl`
``` kotlin
kotlinApplication {
  dependencies {
    implementation(libs.someDependency)
  }

  targets {
    jvm {
      jdkVersion = 17
      mainClass = "com.example.AppKt"
      dependencies {
        implementation(libs.someJvmDependency)
      }
    }
    macOsArm64 {
      entryPoint = "com.example.main"
    }
  }
}
```

The Declarative Gradle example looks familiar, but arbitrary code is not supported. Gradle could therefore statically parse this file. This means that the IDE would be able to improve things like auto-complete, refactoring, and documentation quick access. Imagine the improvements we got when moving from Groovy build files to Kotlin, but even better!

It would even be possible to edit entire build files with a graphical interface, like with android’s xml resource files.
Let’s hope the Android Studio team isn’t reading this...


## Amped for Amper
Can we go even further? In the introduction, we made fun of Python’s requirements.txt system. But what if our project setup was actually simple? Can we skip the confusing `/src/jsMain/kotlin/com/example/my/code` folder structure and have the project setup in a text file?

The Cargo build system for Rust is very approachable and simple to learn. Setting up projects is quick so you can go directly to writing code and arguing with the Rust borrow checker.

Can we do this in Kotlin? That’s the question that the Amper project from JetBrains is trying to answer.

Currently, the Amper project is seen as a build configuration tool only, and the current implementation is built upon Gradle.

The syntax is new, but familiar. Current prototypes use yaml, but no decision has been made on filetype. Apologies if you are reading this in KotlinConf 2024 and just watched the Amper team announce something about requirements.txt

> ### Kotlin application `module.yaml`

``` yaml
product: jvm/app

dependencies:
- org.jetbrains.kotlinx:kotlinx-datetime:0.6.0

test-dependencies:
- org.jetbrains.kotlin:kotlin-test:1.9.0

settings:
  kotlin:
    languageVersion: 1.8
  java:
    source: 17
  jvm:
    target: 17
```

Sure looks like yaml. Also looks nice and simple. Folder structure is different from a typical gradle project as well. Source files live in the `/src` folder, or optionally in e.g. `/src@android` for multiplatform modules. In theory, project and build setup could be drastically simplified by moving to Amper.

## Get to the point
Declarative Gradle promises to improve tooling and speed up builds. It could provide a path to migrating existing projects to a simple but powerful version of Gradle that avoids the footguns of today. “Simple but powerful”. Let's see what the Gradle developers can deliver!

Playing with Amper feels fun and refreshing, and should make it easier to start new projects, especially for new developers. At the same time, Amper is a big departure from the current system and it's not clear if us developers will ever migrate our projects to a new system.

The future of Kotlin build systems is pretty exciting! Just remember that these projects are prototypes, so maybe don't rewrite your 200 module enterprise project to use either.

#### Further reading

 - https://blog.jetbrains.com/blog/2023/11/09/amper-improving-the-build-tooling-user-experience/
 - https://github.com/JetBrains/amper
 - https://github.com/gradle/declarative-gradle/
 - https://doc.rust-lang.org/rust-by-example/cargo.html