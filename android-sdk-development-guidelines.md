# Android SDK Development Guidelines

### Preamble
These are common guidelines for Rakuten teams building Android SDKs/libraries.

#### Terminology:

* The term "library", "SDK" and "SDK module" are used interchangeably in this document. 
* MAY, SHOULD, SHOULD NOT, MUST, MUST NOT, RECOMMENDED are defined in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).
* We distinguish between production code, i.e. code that is exported/visible to clients, and supporting code, i.e. code that is not exported/visible ship to clients, e.g. tests, build scripts, utilities.
* We distinguish exported dependencies, i.e. dependencies that consumers of our modules will have automatically have in their projects as well, and not exported dependencies, i.e. dependencies that are completely hidden from SDK consumers.

## Guidelines
* [Project Files & Source Control Management](#project-files-&-source-control-management)
* [Build](#build)
* [Programming Language](#programming-language)
* [Coding Style](#coding-style)
* [Library Design](#library-design)
* [Testing](#testing)
* [Documentation](#documentation)
* [Versioning](#versioning)
* [Publishing](#publishing)
* [Continuous Integration & Deployment](#continuous-integration-&-deployment)

### Project files & Source Control Management
* MUST be under source control management (SCM). RECOMMENDED source control management is git
* If you intend your source repo to be public ensure you have received the necessary L3/L2 manager and legal department approval
* MUST use a .gitignore configuration
* MUST prefix all exported resources (layout files, xml files, drawable files, color ids, dimension ids, string ids, etc.)  with <shortname> + "_" (e.g. "sug_" for a Suggestion Module) using [resourcePrefix](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.LibraryExtension.html#com.android.build.gradle.LibraryExtension:resourcePrefix) in the gradle build
* SHOULD follow the standard project layout for [Android](https://developer.android.com/studio/projects) or for [Java](https://docs.gradle.org/current/userguide/java_plugin.html#sec:java_project_layout) projects respectively 
* Source code repositories SHOULD follow the naming convention `[platform]-[library name]`, e.g. `android-perftracking`, `java-datastore`, `android-analytics`
    * Rationale: encoding version information (e.g. alpha/core/stable) in the repository breaks links and integration when a module changes
    * Exceptions: existing modules that already carry additional information in the repository name & renaming would break too many existing integrations.
* MUST NOT restrict read access to source code repositories unnecessarily
* For internally hosted code → add the appropriate mailing list with read access
* For open source projects → public repo on GitHub 

### Build
* MUST use [gradle](https://docs.gradle.org/current/userguide/userguide.html) with [gradle wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) to build Android library or Java library
* Development version MUST build with single command after clean clone, i.e. no local setup necessary
* MUST have consistent minSdk and supportLib versions with other SDKs
    * SHOULD use [shared build configuration](https://github.com/rakutentech/android-buildconfig) for consistent version with other SDKs/libraries
    * MAY use manual configuration with values (same as in [shared build configuration](https://github.com/rakutentech/android-buildconfig)), if so 
        * SHOULD use minSdk 21
        * SHOULD use latest versions of [Android X](https://developer.android.com/jetpack/androidx) libraries (if required)
* MUST NOT depend on locally stored libraries 
* MUST NOT use wildcards (+) for exported dependencies, i.e. dependencies that SDK clients will inherit
    * Example 1: using the [auto value](https://github.com/google/auto/tree/master/value) annotation processor in version 1.4-rc1 is allowed because it is not exported to SDK clients
    * Example 2: using [okhttp](https://github.com/square/okhttp) compile/implementatino dependency in version 3.4.0-RC1 is not allowed because that dependency will be imposed onto SDK clients

### Programming Language

#### Production Code

* MUST use Kotlin or Java. Kotlin is RECOMMENDED
* MAY use Java 8 language features under these constraints:
    * MUST be compatible with application projects that do not use [desugar](https://developer.android.com/studio/write/java8-support.html)
    * MAY rely on [retrolambda](https://github.com/orfjackal/retrolambda)
* MAY NOT use other JVM languages (except for Kotlin)
* MAY use native C/C++ when necessary

#### Supporting Code

* MAY use other JVM langages for testing
* SHOULD use gradle/groovy for build scripts
* MAY use any language/framework/tool for other supporting code

### Coding Style

* Code MUST follow the [SDK Kotlin Style Guide](#sdk-kotlin-style-guide) and [SDK Java Style Guide](#sdk-java-style-guide)
* SHOULD use static code analysis, such as [detekt](https://github.com/arturbosch/detekt)
* Android code MUST use the [Android Support Annotations](https://developer.android.com/studio/write/annotations) for public APIs 
* Non-Android Java code MUST use the [Jetbrains Annotations](https://www.jetbrains.com/help/idea/annotating-source-code.html) for public APIs

#### SDK Kotlin Style Guide
* MUST follow [Kotlin's official coding conventions](https://kotlinlang.org/docs/reference/coding-conventions.html)

**Note**: To configure the IntelliJ formatter according to this style guide, please install Kotlin plugin version 1.2.20 or newer, go to Settings | Editor | Code Style | Kotlin, click on "Set from…" link in the upper right corner, and select "Predefined style / Kotlin style guide" from the menu

#### SDK Java Style Guide
* MUST follow [Google Java Style](https://google.github.io/styleguide/javaguide.html)

### Library Design

#### Limit impact on consumers
* MUST use a [consumerProguardFiles](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.BuildType.html#com.android.build.gradle.internal.dsl.BuildType:consumerProguardFiles) file if obfuscation breaks the libraries function 
* SHOULD limit dependencies as much as possible
* SHOULD NOT have exported dependency on other SDK modules
* SHOULD NOT have unnecessary exported dependencies, in particular
    * SHOULD NOT leak networking library to consumers
    * SHOULD NOT leak serialization library to consumers. If unavoidable we recommend [gson](https://github.com/google/gson) for JSON
    * SHOULD use JDK 7 date/time classes and only rely on Java8 Date/Time backport [310bp](https://www.threeten.org/threetenbp/) when absolutely necessary
    * An exception to this rule are optional adapters for popular libraries

#### Expose minimal Surface area
* Every public class, member & resource MUST have a reason for being public
* MUST hide classes that need to be public for technical reasons but are not intended for SDK clients from documentation
* SHOULD declare classes as final to avoid unexpected high-jacking of SDK behavior by subclasses
* SHOULD use RestrictTo annotation on classes that are internal to the library

#### Best Practices
* Simple Configurations SHOULD be [declarable in manifest and read in a (fake) ContentProvider](https://firebase.googleblog.com/2016/12/how-does-firebase-initialize-on-android.html)
* Complex Configurations MUST use the [builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) with [value objects](https://en.wikipedia.org/wiki/Value_object) for configuration values
    * Exception: When using Kotlin [named arguments](https://kotlinlang.org/docs/reference/functions.html#named-arguments) and [default arguments](https://kotlinlang.org/docs/reference/functions.html#default-arguments)

### Testing

* All library (sub-)projects MUST generate coverage test reports (e.g. jacoco, cobertura)
* Unit test coverage MUST be at least 70% and SHOULD be above 80%
* Unit tests MUST NOT rely on reflection to expose otherwise private/internal classes or members

### Documentation

* Public API MUST be annotated with Javadoc or KDoc
    * Exception: when intent is obvious
    * Exception: when class is excluded from javadoc generation
* Libraries MUST publish a self contained user guide
* Libraries MUST publish a changelog with dates of releases
* Libraries MUST contain developer documentation with the source code
* Both user guide and developer documentation SHOULD render readable in the SCM repository web interface (Bitbucket Server/GitHub)
* Versioned User Documentation (= docs + user guide) MUST be published 
    * Internal libs → uploaded to the SDK documentation portal
    * Open Source libs → GitHub pages & KDoc or JavaDoc

### Versioning

* MUST follow [Semantic Versioning](http://semver.org/spec/v2.0.0.html)
* MUST tag source control management revision/commit that is published
* We consider 0.x versions alpha and  1.0 and above stable

### Publishing

Version MUST follow Versioning Guidelines
* Library MUST publish binary as aar or jar to a maven or ivy repository.
* Internal libraries MUST be published in the Mobile Libs Artifactory Repo
* Public libraries SHOULD be published in [jcenter](https://bintray.com/bintray/jcenter)
* Libraries SHOULD publish sources

### Continuous Integration & Deployment

* MUST use continuous integration to ensure quality (e.g. run all tests for all code changes)
* MUST use scm branching + code reviews + continious integrations to ensure quality
    * MUST have code reviewed by at least 1 other engineer before merging
    * MUST run all automated tests without failure before merging
    * MUST address all tasks (created by reviewers) before merging
    * SHOULD integrate CI with code review system
    * SHOULD run checkstyle, android lint, pmd and findbugs on every pull request
* SHOULD automate deployments