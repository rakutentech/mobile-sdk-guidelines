# SDK Development Guidelines

### Preamble
These are common guidelines for Rakuten teams building Android and/or iOS SDKs/libraries.

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
* If you intend your source repo to be public ensure you have received the necessary L3/L2 manager and legal department approval
* MUST use a .gitignore configuration
* Source code repositories SHOULD follow the naming convention `[platform]-[library name]`, e.g. `android-analytics`, `java-datastore`, `ios-inapmessaging`, `swift-utils`
    * Rationale: encoding version information (e.g. alpha/core/stable) in the repository breaks links and integration when a module changes
    * Exceptions: existing modules that already carry additional information in the repository name & renaming would break too many existing integrations.
* MUST NOT restrict read access to source code repositories unnecessarily
* For internally hosted code → add the appropriate mailing list with read access
* For open source projects → public repo on GitHub
* For Android <img src="images/android.png" alt="drawing" width="15"/>:
    * SHOULD follow the standard project layout for [Android](https://developer.android.com/studio/projects) or for [Java](https://docs.gradle.org/current/userguide/java_plugin.html#sec:java_project_layout) projects respectively
    * MUST prefix all exported resources (layout files, xml files, drawable files, color ids, dimension ids, string ids, etc.)  with <shortname> + "_" (e.g. "sug_" for a Suggestion Module) using [resourcePrefix](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.LibraryExtension.html#com.android.build.gradle.LibraryExtension:resourcePrefix) in the gradle build
* For iOS <img src="images/ios.png" alt="drawing" width="15"/>:
    * SHOULD follow the standard project layout for [iOS](https://developer.apple.com/documentation/xcode/managing-files-and-folders-in-your-xcode-project) projects.
    * RECOMMEND to not have source file header comment blocks (such as the Xcode default header comment) as these easily get out of date and the information such as author/version of commits is tracked in SCM


### Build
* Development version MUST build with single command after clean clone, i.e. no local setup necessary
* MUST NOT depend on locally stored libraries
* MUST NOT use wildcards (+) for exported dependencies, i.e. dependencies that SDK clients will inherit
* MUST have a sample app - a minimal app that integrates the SDK and shows how to use the SDK features. It SHOULD be written in Kotlin (Android) or Swift (iOS).
* For Android <img src="images/android.png" alt="drawing" width="15"/>:
    * MUST use [gradle](https://docs.gradle.org/current/userguide/userguide.html) with [gradle wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) to build Android library or Java library
    * MUST have consistent minSdk and supportLib versions with other SDKs
        * SHOULD use [shared build configuration](https://github.com/rakutentech/android-buildconfig) for consistent version with other SDKs/libraries
        * MAY use manual configuration with values (same as in [shared build configuration](https://github.com/rakutentech/android-buildconfig)), if so
            * SHOULD use minSdk 23
            * SHOULD use latest versions of [Android X](https://developer.android.com/jetpack/androidx) libraries (if required)
        * Example 1: using the [auto value](https://github.com/google/auto/tree/master/value) annotation processor in version 1.4-rc1 is allowed because it is not exported to SDK clients
        * Example 2: using [okhttp](https://github.com/square/okhttp) compile/implementatino dependency in version 3.4.0-RC1 is not allowed because that dependency will be imposed onto SDK clients
* For iOS <img src="images/ios.png" alt="drawing" width="15"/>:
    * SHOULD be built from a [CocoaPods](https://guides.cocoapods.org/) Podspec or [SPM](https://www.swift.org/package-manager/)
    * SHOULD use [fastlane](https://docs.fastlane.tools/) to automate tasks and MAY use our [shared build configuration](https://github.com/rakutentech/ios-buildconfig) to help with setup and maintain consistency
    * SHOULD target iOS 12.0 and above, MAY target a more recent iOS version
    * Xcode unit test project SHOULD pull the module's source from a local Podspec path reference
    * Xcode sample project MAY be separate from the unit test project and sample project SHOULD pull the module's source from a local Podspec path reference
    * Xcode project SHOULD build using the latest version of Xcode and the iOS SDK with no warnings

### Programming Language

#### Production Code

* For Android <img src="images/android.png" alt="drawing" width="15"/>:
    * MUST use Kotlin or Java. Kotlin is RECOMMENDED
    * RECOMMENDED to use Java 11 language features
    * MAY NOT use other JVM languages (except for Kotlin)
* For iOS <img src="images/ios.png" alt="drawing" width="15"/>:
    * MUST use Swift or Objective-C. Swift is RECOMMENDED
        * If the SDK needs to execute code before the application is launched you SHOULD use Objective-C (e.g. `+ (void)load`) for at least that part of the code. This is because Swift [does not have](http://jordansmith.io/handling-the-deprecation-of-initialize/) reliable static initializers
* MAY use native C/C++/other languages when necessary

#### Supporting Code

* For Android <img src="images/android.png" alt="drawing" width="15"/>:
    * MAY use other JVM langages for testing
    * SHOULD use gradle/groovy for build scripts
* For iOS <img src="images/ios.png" alt="drawing" width="15"/>:
    * SHOULD use Swift for test code
    * SHOULD use [fastlane](https://docs.fastlane.tools/) (Ruby) for automation but MAY use other languages
* MAY use any language/framework/tool for other supporting code

#### Objective-C Best Practices (iOS <img src="images/ios.png" alt="drawing" width="15"/>):

* If the feature your module provides only works with more recent versions of iOS you MUST do proper feature-checking with e.g. `respondsToSelector:` and still target iOS 10.0 (or more recent version) for the deployment target.
* Nullability: MUST use `NS_ASSUME_NONNULL_BEGIN` and `NS_ASSUME_NONNULL_END` in header files, and explicitly add nullable when needed. The `_Nullable` style syntax is preferred to `__nullable`.
* SHOULD NOT declare any static variable in public headers, as it can cause problems with Cocoapods.
* SHOULD declare a public (and exported) constant and assign its value in a .m source file.
* SHOULD use #pragma mark to logically separate the source code
* [Symbol visibility](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/DynamicLibraryDesignGuidelines.html#//apple_ref/doc/uid/TP40002013-SW18): All classes (public and private) and public constants SHOULD be exported (UIKIT_EXTERN or equivalent). When code is built with `-fvisibility=hidden`, classes not exported by dynamic frameworks don't exist in the ObjC runtime, which is why it's needed on private classes as well. It may be an Apple bug.
    * Note: You do not have to export protocols and categories, notwithstanding their visibility.
* MAY embed private classes in the .m file that uses them, instead of creating a separate .h/.m pair.
* If a value is only ever used once, it's ok to not define a constant for it as long as the intent is clear (e.g. values passed to `+[UColor colorWithRed:green:blue:alpha:])` or documented with a comment.

### Coding Style

* Code MUST follow a style guide such as:
    * [Kotlin's official coding conventions](https://kotlinlang.org/docs/reference/coding-conventions.html)
        * To configure the IntelliJ formatter according to this style guide, please install Kotlin plugin version 1.5.0 or newer, go to Settings | Editor | Code Style | Kotlin, click on "Set from…" link in the upper right corner, and select "Predefined style / Kotlin style guide" from the menu
    * [Google Java Style](https://google.github.io/styleguide/javaguide.html)
    * [Swift](https://github.com/raywenderlich/swift-style-guide)
    * [Objective-C](https://github.com/raywenderlich/objective-c-style-guide)
* Existing modules SHOULD follow a coding style guide. Always strive for **consistency** within a project's codebase
* SHOULD use static code analysis, such as [detekt](https://github.com/arturbosch/detekt), clang-format, [SwiftLint](https://github.com/realm/SwiftLint), or oclint to minimise errors and minimise debates and formatting fix-up commits in PRs. It is RECOMMENDED to integrate these tools with the CI workflow as pre-commit git hooks.
* SHOULD use quality tools available in common config: [Android](https://github.com/rakutentech/android-buildconfig) and [iOS](https://github.com/rakutentech/ios-buildconfig)
* For Android <img src="images/android.png" alt="drawing" width="15"/>:
    * MUST use the [Android Support Annotations](https://developer.android.com/studio/write/annotations) for public APIs
    * Non-Android Java code MUST use the [Jetbrains Annotations](https://www.jetbrains.com/help/idea/annotating-source-code.html) for public APIs

### Library Design

#### Limit impact on consumers
* SHOULD limit dependencies as much as possible primarily due to risk of version clashes e.g. SHOULD NOT depend on external networking library but also because the dependency may stop being maintained
* SHOULD NOT have exported dependency on other SDK modules
* SHOULD NOT have an exported dependency on other SDK modules e.g. if your module needs authentication, do not depend on Authentication SDK. Instead, require your library consumer to pass an access token string to your API instead.
* SHOULD NOT have unnecessary exported dependencies, in particular
    * SHOULD NOT leak networking library to consumers
    * SHOULD NOT leak serialization library to consumers
    * An exception to this rule are optional adapters for popular libraries
* For Android <img src="images/android.png" alt="drawing" width="15"/>:
    * MUST use a [consumerProguardFiles](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.BuildType.html#com.android.build.gradle.internal.dsl.BuildType:consumerProguardFiles) file if obfuscation breaks the libraries function

#### Expose minimal Surface area
* Keep everything private unless it needs to be public. Every public class, method, property & resource MUST have a strong reason for being public
* MAY keep internal source code in a `/Private` folder to allow tools to more easily ignore those files
* MUST hide classes that need to be public for technical reasons but are not intended for SDK clients from documentation
* SHOULD declare classes as final to avoid unexpected high-jacking of SDK behavior by subclasses
* For Android <img src="images/android.png" alt="drawing" width="15"/>:
    * SHOULD use `RestrictTo` annotation on classes that are internal to the library

#### Best Practices
* Simple configuration keys MAY be settable in application's manifest file (Android) or Info.plist (iOS) or passed directly to an API configuration function:
* For Android <img src="images/android.png" alt="drawing" width="15"/>:
    * the manifest file is read in a [(fake) ContentProvider](https://firebase.googleblog.com/2016/12/how-does-firebase-initialize-on-android.html) and be automatically initialized (if that fits with your library design)
    * SHOULD NOT require manual initialization such as by overriding the `Application` class or depending on a host app trigger
    * Complex Configurations MUST use the [builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) with [value objects](https://en.wikipedia.org/wiki/Value_object) for configuration values
        * Exception: When using Kotlin [named arguments](https://kotlinlang.org/docs/reference/functions.html#named-arguments) and [default arguments](https://kotlinlang.org/docs/reference/functions.html#default-arguments)
    * For better testability of your library it is RECOMMENDED to use [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)
* For iOS <img src="images/ios.png" alt="drawing" width="15"/>:
    * Take the https://swift.org/documentation/api-design-guidelines/ into account when designing your public API interface

### Testing

* All library (sub-)projects MUST generate coverage test reports (e.g. jacoco, xcode/slather)
* Unit test coverage MUST be at least 80% and SHOULD be above 85%
* For Android <img src="images/android.png" alt="drawing" width="15"/>:
    * Unit tests MUST NOT rely on reflection to expose otherwise private/internal classes or members
    * Tests SHOULD use JUnit, and MAY use helper libraries such as Mockito or Robolectric
* For iOS <img src="images/ios.png" alt="drawing" width="15"/>:
    * Tests SHOULD use XCTest, and MAY use helper libraries such as OCMock or OHHTTPStubs
    * Tests MAY be BDD style using Kiwi for Obj-C or Quick/Nimble for Swift

### Documentation

* Public API MUST be annotated with a standard documenting style e.g. Doxygen, Appledoc, Javadoc or KDoc
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
    * RECOMMENDED for Android <img src="images/android.png" alt="drawing" width="15"/>: Use [gradle-versions-plugin](https://github.com/ben-manes/gradle-versions-plugin) for automated versioning by git tag
* We consider 0.x versions alpha and  1.0 and above stable

### Publishing

* Version MUST follow Versioning guide above
* Libraries SHOULD publish sources internally or to GitHub if open sourced
* For Android <img src="images/android.png" alt="drawing" width="15"/>:
    * Library MUST publish binary as aar or jar to a maven or ivy repository.
    * Internal libraries MUST be published in the Mobile Libs Artifactory Repo
    * Public libraries SHOULD be published in [Maven Central](https://mvnrepository.com/repos/central)
* For iOS <img src="images/ios.png" alt="drawing" width="15"/>:
    * Library SHOULD publish Podspec or SPM
    * Internal libraries that use CocoaPods SHOULD publish their Podspec in the MAG spec repo (fork and create PR)
    * Public libraries that use CocoaPods SHOULD be published to CocoaPods master repo
    * Libraries MAY publish a Podspec that points to a framework binary and keep their source private

### Continuous Integration & Deployment

* MUST use scm branching + code reviews + continious integrations to ensure quality (e.g. run all tests for all code changes)
    * PRs MUST be approved after code review by at least 2 other code owners before merging
    * PRs MUST run all automated tests without failure before merging
    * PRs MUST address all tasks/comments (created by reviewers) before merging
    * PRs SHOULD run all quality checks
* SHOULD integrate CI with code review system
* SHOULD use automated deployments