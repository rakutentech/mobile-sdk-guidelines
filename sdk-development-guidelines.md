# SDK Development Guidelines

### Preamble
These are common guidelines for Rakuten teams building Android and/or iOS SDKs/libraries.

#### Contribution
**We encourage everyone to create PR contributions for additional guidelines, or create issues for any discussion points.**

#### Terminology:

* The term "library", "SDK", "module" and "SDK module" are used interchangeably in this document.
* MAY, SHOULD, SHOULD NOT, MUST, MUST NOT, RECOMMENDED are defined in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).
* We distinguish between **production code**, i.e. code that is exported/visible to clients, and **supporting code**, i.e. code that is not exported/visible to clients, e.g. tests, build scripts, utilities.
* We distinguish between **exported dependencies**, i.e. dependencies that consumers of our modules will automatically get in their projects as well, and **not exported dependencies**, i.e. dependencies that are completely hidden from SDK consumers.

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
* If you intend your source repo to be public ensure you have received the necessary management and legal approval
* MUST use a .gitignore configuration
* Source code repositories SHOULD follow the naming convention `[platform]-[library name]`, e.g. `android-analytics`, `java-datastore`, `ios-inapmessaging`, `swift-utils`
    * Rationale: encoding version information (e.g. alpha/core/stable) in the repository breaks links and integration when a module changes
    * Exceptions: existing modules that already carry additional information in the repository name & renaming would break too many existing integrations.
* MUST NOT restrict read access to source code repositories unnecessarily
* For internally hosted code → add the appropriate mailing list with read access
* For open source projects → public repo on GitHub
* RECOMMEND to not have source file header comment blocks (such as the Xcode/Java default header comment) as these easily get out of date and the information such as author/version of commits is tracked in SCM
* <img src="images/android.png" alt="drawing" width="15"/> For Android:
    * SHOULD follow the standard project layout for [Android](https://developer.android.com/studio/projects) or for [Java](https://docs.gradle.org/current/userguide/java_plugin.html#sec:java_project_layout) projects respectively
    * MUST prefix all exported resources (layout files, xml files, drawable files, color ids, dimension ids, string ids, etc.)  with <shortname> + "_" (e.g. "sug_" for a Suggestion Module) using [resourcePrefix](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.LibraryExtension.html#com.android.build.gradle.LibraryExtension:resourcePrefix) in the gradle build
* <img src="images/ios.png" alt="drawing" width="15"/> For iOS:
    * SHOULD follow the standard project layout for [iOS](https://developer.apple.com/documentation/xcode/managing-files-and-folders-in-your-xcode-project) projects.

### Build
* Development version MUST build with single command after clean clone, i.e. no local setup necessary
* MUST NOT depend on locally stored libraries
* MUST NOT use wildcards (+) for exported dependencies, i.e. dependencies that SDK clients will inherit
* MUST have a sample app - a minimal app that integrates the SDK and shows how to use the SDK features. It SHOULD be written in Kotlin (Android) or Swift (iOS).
* <img src="images/android.png" alt="drawing" width="15"/> For Android:
    * MUST use [gradle](https://docs.gradle.org/current/userguide/userguide.html) with [gradle wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) to build Android library or Java library
    * MUST have consistent minSdk and supportLib versions with other SDKs
        * SHOULD use [shared build configuration](https://github.com/rakutentech/android-buildconfig) for consistent version with other SDKs/libraries
        * MAY use manual configuration with values (same as in [shared build configuration](https://github.com/rakutentech/android-buildconfig)), if so
            * SHOULD use a recent minSdk version unless there is an engineering or product decision to support older versions
            * SHOULD use latest versions of [Android X](https://developer.android.com/jetpack/androidx) libraries (if required)
        * Example 1: using the [auto value](https://github.com/google/auto/tree/master/value) annotation processor in version 1.4-rc1 is allowed because it is not exported to SDK clients
        * Example 2: using [okhttp](https://github.com/square/okhttp) compile/implementatino dependency in version 3.4.0-RC1 is not allowed because that dependency will be imposed onto SDK clients
* <img src="images/ios.png" alt="drawing" width="15"/> For iOS:
    * SHOULD be built from a [CocoaPods](https://guides.cocoapods.org/) Podspec or [Swift Package Manager (SPM)](https://www.swift.org/package-manager/)
    * SHOULD use [fastlane](https://docs.fastlane.tools/) to automate tasks and MAY use our [shared build configuration](https://github.com/rakutentech/ios-buildconfig) to help with setup and maintain consistency
    * SHOULD set minimum deployment target to a recent iOS version unless there is an engineering or product decision to support older versions
    * Xcode unit test project SHOULD pull the module's source from a local Podspec path reference
    * Xcode sample project MAY be separate from the unit test project and sample project SHOULD pull the module's source from a local Podspec path reference
    * Xcode project SHOULD build using the latest version of Xcode and the iOS SDK with no warnings

### Programming Language

#### Production Code

* <img src="images/android.png" alt="drawing" width="15"/> For Android:
    * MUST use Kotlin or Java. Kotlin is RECOMMENDED
    * RECOMMENDED to use Java 11 language features
    * MAY NOT use other JVM languages (except for Kotlin)
* <img src="images/ios.png" alt="drawing" width="15"/> For iOS:
    * MUST use Swift or Objective-C. Swift is RECOMMENDED
        * If the SDK needs to execute code before the application is launched you MAY use Objective-C (e.g. `+ (void)load`) for at least that part of the code. This is because Swift [does not have](http://jordansmith.io/handling-the-deprecation-of-initialize/) reliable static initializers
* MAY use native C/C++/other languages when necessary

#### Supporting Code

* <img src="images/android.png" alt="drawing" width="15"/> For Android:
    * MAY use other JVM langages for testing
    * SHOULD use gradle (groovy or kotlin) for build scripts
* <img src="images/ios.png" alt="drawing" width="15"/> For iOS:
    * SHOULD use Swift for test code
    * SHOULD use [fastlane](https://docs.fastlane.tools/) (Ruby) for automation but MAY use other languages
* MAY use any language/framework/tool for other supporting code

### Coding Style

* Code MUST follow a style guide such as:
    * [Kotlin's official coding conventions](https://kotlinlang.org/docs/reference/coding-conventions.html)
        * To configure the IntelliJ formatter according to this style guide, please install Kotlin plugin version 1.5.0 or newer, go to Settings | Editor | Code Style | Kotlin, click on "Set from…" link in the upper right corner, and select "Predefined style / Kotlin style guide" from the menu
    * [Google Java Style](https://google.github.io/styleguide/javaguide.html)
    * [Swift](https://github.com/raywenderlich/swift-style-guide)
    * [Objective-C](https://github.com/raywenderlich/objective-c-style-guide)
* Existing modules SHOULD follow a coding style guide. Always strive for **consistency** within a project's codebase
* SHOULD use static code analysis, such as [detekt](https://github.com/arturbosch/detekt) or [SwiftLint](https://github.com/realm/SwiftLint) to minimise errors and minimise debates and formatting fix-up commits in PRs. It is RECOMMENDED to integrate these tools with the CI workflow.
* SHOULD use quality tools available in common config: [Android](https://github.com/rakutentech/android-buildconfig) and [iOS](https://github.com/rakutentech/ios-buildconfig)
* <img src="images/android.png" alt="drawing" width="15"/> For Android:
    * MUST use the [Android Support Annotations](https://developer.android.com/studio/write/annotations) for public APIs
    * Non-Android Java code MUST use the [Jetbrains Annotations](https://www.jetbrains.com/help/idea/annotating-source-code.html) for public APIs

### Library Design

#### Limit impact on consumers
* SHOULD limit dependencies as much as possible primarily due to risk of version clashes (e.g. SHOULD NOT depend on external networking library) but also because the dependency may stop being maintained
* RECOMMENDED not to have any exported dependency on other SDK modules e.g. if your module needs authentication, do not depend on Authentication SDK. Instead, require your library consumer to pass an access token string to your API instead.
* <img src="images/android.png" alt="drawing" width="15"/> For Android:
    * MUST use a [consumerProguardFiles](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.BuildType.html#com.android.build.gradle.internal.dsl.BuildType:consumerProguardFiles) file if obfuscation breaks the libraries function
    * SHOULD NOT have unnecessary exported dependencies, in particular
        * SHOULD NOT leak networking library to consumers
        * SHOULD NOT leak serialization library to consumers
        * An exception to this rule are optional adapters for popular libraries

#### Expose minimal Surface area
* Keep everything private unless it needs to be public. Every public class, method, property & resource MUST have a strong reason for being public
* MAY keep internal source code in a `/Private` folder to allow tools to more easily ignore those files
* MUST hide classes from generated API documentation that are required to be public solely for technical reasons and are not intended for SDK consumers
* SHOULD declare classes as final to avoid unexpected high-jacking of SDK behavior by subclasses
* <img src="images/android.png" alt="drawing" width="15"/> For Android:
    * SHOULD use `RestrictTo` annotation on classes that are internal to the library

#### Best Practices
* For better testability of your library it is RECOMMENDED to use [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)
* Simple configuration keys MAY be settable in application's manifest file (Android) or Info.plist (iOS) or passed directly to an API configuration function:
* <img src="images/android.png" alt="drawing" width="15"/> For Android:
    * the manifest file is read in a [(fake) ContentProvider](https://firebase.googleblog.com/2016/12/how-does-firebase-initialize-on-android.html) and be automatically initialized (if that fits with your library design)
    * Complex Configurations MUST use the [builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) with [value objects](https://en.wikipedia.org/wiki/Value_object) for configuration values
        * Exception: When using Kotlin [named arguments](https://kotlinlang.org/docs/reference/functions.html#named-arguments) and [default arguments](https://kotlinlang.org/docs/reference/functions.html#default-arguments)
* <img src="images/ios.png" alt="drawing" width="15"/> For iOS:
    * Take the https://swift.org/documentation/api-design-guidelines/ into account when designing your public API interface

### Testing

* All library (sub-)projects MUST generate coverage test reports (e.g. jacoco, Xcode/slather)
* Unit test coverage MUST be at least 80% and SHOULD be above 85%
* <img src="images/android.png" alt="drawing" width="15"/> For Android:
    * Unit tests MUST NOT rely on reflection to expose otherwise private/internal classes or members
    * Tests SHOULD use JUnit, and MAY use helper libraries such as Mockito or Robolectric
* <img src="images/ios.png" alt="drawing" width="15"/> For iOS:
    * Tests SHOULD use XCTest, and MAY use helper libraries such as OCMock or OHHTTPStubs
    * Tests MAY be BDD style using Kiwi for Obj-C or Quick/Nimble for Swift

### Documentation

* Public API MUST be annotated with a standard documenting style e.g. Appledoc, Javadoc or KDoc
    * Exception: when intent is obvious
    * Exception: when class is excluded from document generation
* Libraries MUST publish a self contained user guide
* Libraries MUST publish a changelog with dates of releases. If you can't link to an issue tracker due to privacy/access rights you should include a full description of the change.
* Libraries MUST contain developer documentation with the source code
* Both user guide and developer documentation SHOULD render readable in the SCM repository web interface (Bitbucket Server/GitHub)
* Versioned User Documentation (e.g. docs + user guide) MUST be published and SHOULD use a script to automate publishing
    * Internal libs → internal GitHub pages
    * Open Source libs → public GitHub pages

### Versioning

* MUST follow [Semantic Versioning](http://semver.org/spec/v2.0.0.html)
* MUST tag source control management revision/commit that is published
    * RECOMMENDED <img src="images/android.png" alt="drawing" width="15"/> For Android: Use [gradle-versions-plugin](https://github.com/ben-manes/gradle-versions-plugin) for automated versioning by git tag
* We consider 0.x versions alpha and 1.0 and above stable

### Publishing

* Version MUST follow Versioning guide above
* Libraries SHOULD publish sources internally or to GitHub if open sourced
* <img src="images/android.png" alt="drawing" width="15"/> For Android:
    * Library MUST publish binary as aar or jar to a maven or ivy repository.
    * Internal libraries MUST be published in the Mobile Libs Artifactory Repo
    * Public libraries SHOULD be published in [Maven Central](https://mvnrepository.com/repos/central)
* <img src="images/ios.png" alt="drawing" width="15"/> For iOS:
    * Library SHOULD publish Podspec or Swift Package Manager (SPM) manifest
    * Internal libraries that use CocoaPods SHOULD publish their Podspec in the MAG spec repo (fork and create PR)
    * Public libraries that use CocoaPods SHOULD be published to CocoaPods master repo
    * Libraries MAY publish a Podspec that points to a framework binary and keep their source private

### Continuous Integration & Deployment

* MUST use scm branching + code reviews + continious integrations to ensure quality (e.g. run all tests for all code changes)
    * PRs MUST be approved after code review by at least 1 code owner before merging
    * PRs MUST run all automated tests without failure before merging
    * PRs MUST address all tasks/comments (created by reviewers) before merging
    * PRs SHOULD run all quality checks
* SHOULD integrate CI with code review system
* SHOULD use automated deployments