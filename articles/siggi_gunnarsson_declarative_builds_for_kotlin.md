# Declarative builds for Kotlin

### Alternative title: Declarative Gradle? Amper? What’s going on?!

We Kotlin developers have it pretty good as it comes to configuring and executing our builds. We usually work with nice IDEs that help setting projects up for us, and a pretty robust build system in Gradle. Gradle gives us a nice way to import dependencies, compile and package the project. It even helps with running tests as a bonus! 

Not all languages have such a robust and "standardized" system. In Python, dependencies are declared by writing them into a `requirements.txt` file. Then you use `pip` to install everything into your (hopefully virtual) python environment and hope there are no dependency conflicts. Since there will be conflicts, you can write dependency versions into this `.txt` file based on the complaints from `pip` for a while before deciding maybe AI isn't for you.

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
    implementation(libs.someDependency) // Actually comes from a yaml file, but let's keep things simple
}

java {
    toolchain {
        languageVersion.set(JavaLanguageVersion.of(8)) // Java 8 is good, right?
    }
}

tasks.register("funTask") {
    mContext++
    compileCode()
    GlobalScope.launch {
        while (true){
            delay(100)
            println("Hi there :)")
        }
    }
}    

application {
    mainClass.set("MainClassFile")
}
```

As nice and flexible as it is to be able to write build configurations in our favourite programming language, that comes at a cost. When we write custom build logic in our build files, Gradle is not able to statically analyze our build files. This limits how much our IDEs can help us, and can impact build configuration and build times.

We can help by following best practices:
 - Only put declarative software definitions into build scripts
 - Put all your custom build logic into Gradle plugins

Unfortunately, there’s nothing preventing you from writing tasks in an inefficient way. There's a large amount of projects out there where gnarly build logic is peppered around the build scripts. It's also not always obvious if your "declarative" build code really is that. Did your task correctly declare inputs and outputs? Was your new gradle code a "software definition" or did you actually write "build logic"? Did you spot the build logic cleverly hidden in the above example?

### Enter “Declarative Gradle”

The good folks at Gradle are aware of this, and have started work on a new kind of gradle script file. One that only supports limited subset of build configuration functionality. This, in theory, will ensure that build logic is safely tucked away in .kt files in your build plugins. Although this work is still in a prototyping phase, let's look at an example of a kotlin multiplatform application that can run on the JVM and Mac. 


> #### Kotlin application `build.gradle.dcl`
``` kotlin
kotlinApplication {
    dependencies {
        implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.6.0")
    }

    targets {
        jvm {
            jdkVersion = 17
            mainClass = "com.example.AppKt"
            dependencies {
                implementation("org.apache.commons:commons-lang3:3.14.0")
            }
        }
        macOsArm64 {
            entryPoint = "com.example.main"
        }
    }
}
```

In this file format, it's not possible to write arbitary code. Since `.dcl files are fully declarative, Gradle could easily and quickly parse them. The IDE would be able to improve things like auto-complete, refactoring and documentation quick access. Think moving from Groovy build files to Kotlin, but more!. It would even be possible to edit entire build files with a graphical interface, like with android’s xml resource files. 

### Amper

Can we go even further? In the introduction, Python’s `requirements.txt` file got some criticism. But what if our project setup was actually simple? If build configuration was just a text file? Can we skip the `/src/jsMain/kotlin/com/example/my/code` folder structure? 

This can be done. The Cargo build system for Rust is very approachable and simple to learn. Setting up projects is a breeze, and you can go directly to writing code and arguing with the Rust borrow checker.

Would this work in Kotlin? That’s the question that the Amper project from JetBrains is trying to answer. 

Currently, the Amper project is seen as a build configuration tool only, and the current implementation is built upon Gradle. 

The syntax is new, but familiar. Current prototypes use yaml, but no decision has been made on filetype. (Unless you are reading this in KotlinConf 2024 and just watched an Amper talk)

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

Source files are put directly in `/src` folder, or optionally in e.g. a `/src@android` for multiplatform modules. In theory, projects and build setup could be drastically simplified by moving to Amper.

### Get to the point

"Declarative Gradle" promises to improve tooling and speed up builds. If it's ever completed, we would have a path of migrating projects to a simpler, but equally powerful version of Gradle. Let's see if the Gradle developers can actually deliver "simple". 

Playing with Amper feels fun and refreshing, and will make it easier to start new projects, especially for new developers. However, Amper is a radical departure from the current system so it's not clear if it will be a common way of working in the future.

In either case, both projects are prototypes, so maybe don't rewrite your 200 module enterprise project right away.


#### Further reading

 - https://blog.jetbrains.com/blog/2023/11/09/amper-improving-the-build-tooling-user-experience/
 - https://github.com/JetBrains/amper
 - https://github.com/gradle/declarative-gradle/
 - https://doc.rust-lang.org/rust-by-example/cargo.html