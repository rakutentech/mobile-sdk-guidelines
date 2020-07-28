# iOS SDK Development Guidelines

### Preamble
These are common guidelines for Rakuten MTSD (and project partner) teams building iOS SDKs/libraries.

#### Terminology:

* The term "library", "SDK", "module" and "SDK module" are used interchangeably in this document. 
* MAY, SHOULD, SHOULD NOT, MUST, MUST NOT, RECOMMENDED are defined in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).
* We distinguish between **production code**, i.e. code that is exported/visible to clients, and **supporting code**, i.e. code that is not exported/visible to clients, e.g. tests, build scripts, utilities.
* We distinguish between **exported dependencies**, i.e. dependencies that consumers of our modules will automatically get in their projects as well, and **not exported dependencies**, i.e. dependencies that are completely hidden from SDK consumers.

## Guidelines

* [Project files & source control management](#project-files-&-source-control-management)
* [Build](#build)
* [Programming language](#programming-language)
* [Objective-C best practices](#objective-c-best-practices)
* [Library design](#library-design)
* [Testing](#testing)
* [Documentation](#documentation)
* [Versioning](#versioning)
* [Publishing](#publishing)
* [Continuous Integration & Deployment](#continuous-integration-&-deployment)

### Project Files & Source Control Management
* MUST be under source control management such as internal GitPub or GHE. RECOMMENDED source control management is git
* SHOULD use GitPub or GHE for internal-only libraries
* If you intend your source repo to be public ensure you have received the necessary MTSD and legal department permissions to open source your library. If unsure check the Open Source Initiative page on DRC
* MUST use a .gitignore configuration
* RECOMMEND to not have source file header comment blocks (such as the Xcode default header comment) as these easily get out of date and the information such as author/version of commits is tracked in SCM
* Source code repositories SHOULD follow the naming convention [platform]-[library name], e.g. ios-perftracking, swift-datastore, ios-analytics
    * Rationale: encoding version information (e.g. alpha/core/stable) in the repository breaks links and integration when a module changes
    * Exceptions: existing modules that already carry additional information in the repository name & renaming would break too many existing integrations

### Build
* SHOULD be built from a [CocoaPods](https://guides.cocoapods.org/) Podspec (or use a similar package manager like Carthage or SPM)
* SHOULD use [fastlane](https://docs.fastlane.tools/) to automate tasks
* SHOULD target iOS 10.0 and above
* Xcode unit test project SHOULD pull the module's source from a local Podspec path reference
* Xcode sample project MAY be separate from the unit test project and sample project SHOULD pull the module's source from a local Podspec path reference
* Xcode project SHOULD build using the latest version of Xcode and the iOS SDK with no warnings

### Programming language

#### Production Code:

* SHOULD use Swift or Objective-C
    * Swift is preferred
    * If the SDK needs to execute code before the application is launched you SHOULD use Objective-C (e.g. `+ (void)load`) for at least that part of the code. This is because Swift [does not have](http://jordansmith.io/handling-the-deprecation-of-initialize/) reliable static initializers
* New modules MUST follow a style guide such as default SwiftLint configuration, Ray Wenderlich's [Objective-C](https://github.com/raywenderlich/objective-c-style-guide) or [Swift](https://github.com/raywenderlich/swift-style-guide) guides or the [GitHub](https://github.com/github/swift-style-guide) Swift style guide if the team does not have its own internal guide. Existing modules SHOULD follow a coding style guide. Always strive for **consistency** within a project's codebase.
* Take the https://swift.org/documentation/api-design-guidelines/ into account when designing your public API interface
* SHOULD use tools such as clang-format, SwiftLint, and oclint to minimise errors and reduce arguments on PRs and formatting fix-up commits. It is RECOMMENDED to integrate these tools with the CI workflow as pre-commit git hooks.
* MAY use C/C++/other languages when necessary

#### Supporting Code:

* SHOULD use Swift or Objective-C for test code
* SHOULD use fastlane (Ruby) for automation scripts but MAY use other languages
* MAY use any language/framework/tool for other supporting code

### Objective-C best practices
* If the feature your module provides only works with more recent versions of iOS you MUST do proper feature-checking with e.g. `respondsToSelector:` and still target iOS 10.0 for the deployment target.
* Nullability: MUST use NS_ASSUME_NONNULL_BEGIN and NS_ASSUME_NONNULL_END in header files, and explicitly add nullable when needed. The _Nullable style syntax is preferred to __nullable.
* SHOULD NOT declare any static variable in public headers, as it can cause problems with Cocoapods. 
* SHOULD declare a public (and exported) constant and assign its value in a .m source file.
* SHOULD use #pragma mark to logically separate the source code
* [Symbol visibility](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/DynamicLibraryDesignGuidelines.html#//apple_ref/doc/uid/TP40002013-SW18): All classes (public and private) and public constants SHOULD be exported (UIKIT_EXTERN or equivalent). When code is built with -fvisibility=hidden, classes not exported by dynamic frameworks don't exist in the ObjC runtime, which is why it's needed on private classes as well. It may be an Apple bug.
    * Note you do not have to export protocols and categories, notwithstanding their visibility.
* MAY embed private classes in the .m file that uses them, instead of creating a separate .h/.m pair.
* If a value is only ever used once, it's ok to not define a constant for it as long as the intent is clear (e.g. values passed to +[UColor colorWithRed:green:blue:alpha:]) or documented with a comment.

### Library design

#### Limit impact on consumers
* SHOULD limit dependencies as much as possible primarily due to risk of version clashes e.g. SHOULD NOT depend on external networking library but also because the dependency may stop being maintained
* SHOULD NOT have an exported dependency on other SDK modules e.g. if your module needs authentication, do not depend on Authentication SDK. Instead, require your library consumer to pass an access token string to your API instead

#### Expose minimal surface area
* Keep everything private unless it needs to be public. Every public class, method, property & resource MUST have a strong reason for being public
* MAY keep internal source code in a `/Private` folder to allow tools to more easily ignore those files 

#### Configuration options
* Simple configuration keys MAY be settable from an application's Info.plist or passed directly to an API configuration function

### Testing
* All library (sub-)projects MUST generate coverage test reports (e.g. cobertura)
* Unit test coverage MUST be at least 70% and SHOULD be above 80%
* Tests SHOULD use XCTest, and MAY use helper libraries such as OCMock or OHHTTPStubs
* Tests MAY be BDD style using Kiwi for Obj-C or Quick/Nimble for Swift

### Documentation
* Public API MUST be annotated with a standard documenting style e.g. Doxygen, Appledoc
    * Exception: when intent is obvious
    * The internal Objective-C SDK modules currently use Doxygen style and the doxygen tool to generate their portal documentation
* Libraries MUST publish a self contained user guide
* Libraries MUST publish a changelog with dates of releases. If you can't link to an issue tracker due to privacy/access rights you should include a full description of the change.
* Libraries MUST contain developer documentation with the source code
* Both user guide and developer documentation SHOULD render readable in the SCM repository web interface (Bitbucket server/GitHub)
* Versioned User Documentation (e.g. doxygen + user guide) MUST be published and SHOULD use a script to automate publishing
    * Internal libs → deploy to the SDK documentation portal
    * Open Source libs → deploy to GitHub pages

### Versioning
* MUST follow [Semantic Versioning](http://semver.org/spec/v2.0.0.html)
* MUST tag the source control management revision/commit that is published
* We consider 0.x versions alpha and 1.0 and above stable

### Publishing
* Version MUST follow Versioning guide above
* Library SHOULD publish Podspec
* Internal libraries that use CocoaPods SHOULD publish their Podspec in the MTSD spec repo (fork and create PR)
* Public libraries SHOULD be published to CocoaPods master repo 
* Libraries SHOULD publish sources internally or to GitHub if open sourced
* Libraries MAY publish a Podspec that points to a framework binary and keep their source private

### Continuous Integration & Deployment
* MUST use scm branching + code reviews + continuous integration to ensure quality (e.g. run all tests for all code changes)
    * PRs MUST be approved after code review by at least 1 other engineer before merging
    * PRs MUST pass all automated tests before merging
    * PRs MUST address all tasks/comments (created by reviewers) before merging
* SHOULD integrate CI with code review system
* SHOULD use automated deployments
