# Xcode Project Settings

- [Xcode Project Settings](#xcode-project-settings)
  - [iOS Keys for Info.plist](#ios-keys-for-infoplist)
  - [Build settings](#build-settings)
  - [Dependencies](#dependencies)
    - [Unused Dependencies](#unused-dependencies)
      - [Dead Code Elimination / Link-Time Optimization (LTO)](#dead-code-elimination--link-time-optimization-lto)
      - [App Store Submission and App Thinning](#app-store-submission-and-app-thinning)
      - [Dynamic Frameworks and Lazy Loading](#dynamic-frameworks-and-lazy-loading)
  - [Scheme vs Configuration](#scheme-vs-configuration)
    - [Scheme](#scheme)
    - [Configuration](#configuration)
  - [DEBUG/DEVELOPMENT Configurations Name for SPM](#debugdevelopment-configurations-name-for-spm)
  - [dSYM](#dsym)

## iOS Keys for Info.plist

The iOS frameworks provide the infrastructure you need for creating iOS apps.
You use the keys associated with this framework to configure the appearance of
your app at launch time and the behavior of your app once it is running.

[List of iOS keys for Info.plist](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/uid/TP40009252-SW1)

## Build settings

A detailed list of individual Xcode build settings that control or change the
way a target is built.

[List of build settings for Xcode projects](https://developer.apple.com/documentation/xcode/build-settings-reference)

## Dependencies

Using the following command line, it's possible to get a list of framework that
the binary depends on (based on [this post](https://developer.apple.com/forums/thread/655588?answerId=712400022#712400022)
from Quinn):

```bash
xcrun dyld_info -dependents <path-to-binary>
```

For instance:

```text
.../DerivedData/MyApp/Build/Products/Development-iphonesimulator/MyFramework.framework/MyFramework [arm64]:
    -linked_dylibs:
        attributes     load path
                       @rpath/Apollo.framework/Apollo
                       @rpath/ApolloAPI.framework/ApolloAPI
                       @rpath/FraudForce.framework/FraudForce
                       /System/Library/Frameworks/Foundation.framework/Foundation
        weak-link      /usr/lib/swift/libswiftXPC.dylib
        weak-link      /usr/lib/swift/libswift_Builtin_float.dylib
                       /usr/lib/swift/libswift_Concurrency.dylib
                       /usr/lib/swift/libswiftos.dylib
        weak-link      /usr/lib/swift/libswiftsimd.dylib
        weak-link      /usr/lib/swift/libswiftUIKit.dylib
```

It's also possible to use [otool](https://llvm.org/docs/CommandGuide/llvm-otool.html)
to list the dependencies used by a binary:

```basd
xcrun otool -L <path-to-binary>
```

The output is going to be slightly different from what `dyld_info` yields:

```bash
/Users/MyUser/Library/Developer/Xcode/DerivedData/MyApp/Build/Products/Development-iphonesimulator/MyFramework.framework/MyFramework:
  @rpath/MyFramework.framework/MyFramework (compatibility version 1.0.0,
  current version 1.0.0)
  @rpath/MyOtherFramework.framework/MyFramework (compatibility version 1.0.0,
  current version 1.0.0)
```

Note that `@rpath` is for creating a run-path dependent library. Based on [the documentation](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/RunpathDependentLibraries.html):

> You specify a run-path–relative pathname as the library’s install name.
A run-path-relative pathname uses the `@rpath` macro to specify a path
relative to a directory to be determined at runtime.

In other words, the `@rpath` in dynamic linking refers to a runtime search path
mechanism used by the dynamic linker to locate the frameworks or dynamic
libraries required by an app. This is the opposite of hardcoding the path.

The `@rpath` placeholder is resolved by the dynamic loader (`dyld`) using the
runtime search paths provided at link time, such as via the `-rpath` option.
These paths can include locations like the directory of the app bundle or
standard system directories.

### Unused Dependencies

Unused dependencies are typically not automatically removed by default.
When dealing with dependencies, there are two items in *Build Settings* to look after:

1. [LIBRARY_SEARCH_PATHS](https://developer.apple.com/documentation/xcode/build-settings-reference#Library-Search-Paths):
This is a list of paths to folders to be searched by the linker for libraries
used by the product. Paths are delimited by whitespace, so any paths with spaces
in them need to be properly quoted.
2. [FRAMEWORK_SEARCH_PATHS](https://developer.apple.com/documentation/xcode/build-settings-reference#Framework-Search-Paths):
This is a list of paths to folders containing frameworks to be searched by
the compiler for both included or imported header files when compiling C,
Objective-C, C++, or Objective-C++, and by the linker for frameworks used
by the product. Paths are delimited by whitespace, so any paths with spaces
in them must be properly quoted.

Here’s a breakdown of how unused dependencies might be handled:

#### Dead Code Elimination / Link-Time Optimization (LTO)

Xcode can remove unused code through *Dead Code Elimination (DCE)* or
*Link-Time Optimization (LTO)*, but this primarily focuses on code within
the same binary. These processes can reduce the size of the executable by
stripping out unused symbols and functions, but they do not remove
entire frameworks or libraries**.

- *LTO* optimizes code during the linking phase, ensuring that any unused
symbols are stripped from the final binary.
- *DCE* ensures that any functions or code paths that aren’t referenced
are removed from the executable.

These optimizations are set in the *Build Settings* under
`Enable Link Time Optimization` and `Dead Code Stripping`.

Unused frameworks that are still linked (but not used) will not be automatically
removed from the project by any standard build process. They will still be
included in the build output unless specifically unlinked or excluded manually.

- *Manual Removal:* You can manually unlink frameworks that are no longer
needed via Xcode by going to the *Build Phases* of your target, under
*Link Binary With Libraries*, and removing the unused frameworks.

#### App Store Submission and App Thinning

During the *App Store submission* process, *App Thinning* helps reduce the app
size by delivering only the assets and code needed for the target device.
However, this won’t affect unused linked dependencies– those still need to be
managed at the project level before submission.

#### Dynamic Frameworks and Lazy Loading

If you are using [dynamic frameworks](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html)
, even if they are listed as a dependency, they will only be loaded into memory
when they are actually used (thanks to *lazy loading*). However, the framework
itself will still be included in the app bundle unless explicitly removed.

In summary, unused dependencies won’t be removed automatically by the build
system, but the app size may be optimized if you enable DCE and LTO. For
complete removal of an unused dependency, it has to be manually unlinked
from the project configuration.

Concepts to look into later:

- [Frameworks and Weak Linking](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/WeakLinking.html)
- [Frameworks and Binding](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/FrameworkBinding.html)
- [Dynamic Library Design Guidelines](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/DynamicLibraryDesignGuidelines.html)
- [Executing Mach-O Files](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/MachOTopics/1-Articles/executing_files.html#//apple_ref/doc/uid/TP40001829)
- [Creating a static framework](https://developer.apple.com/documentation/xcode/creating-a-static-framework/)

## Scheme vs Configuration

In Xcode, a "scheme" and a "configuration" are two distinct concepts that play
different roles in the build and run processes of an Xcode project.

### Scheme

A scheme is a collection of settings that specify how Xcode
should build, run, test, or profile your app. It includes information such as
which target to build, which build configuration to use, and how to run or test
the app<sup>[*](https://developer.apple.com/documentation/xcode/customizing-the-build-schemes-for-a-project)</sup>.

- **Usage:**
  - Schemes are used to define various actions that can be performed on an Xcode
  project, such as building, running, testing, profiling, or archiving.
  - Each scheme can be associated with one or more build configurations, allowing
  you to specify different settings for different purposes (e.g., Debug, Release).

- **Customization:** You can customize a scheme to perform specific tasks, such
  as running specific tests, launching the app with specific arguments, or specifying
  different environments for development, testing, and production.

- **Common Scenarios:** You might have different schemes for running your app
  locally, running UI or unit tests, or creating an archive for distribution.

### Configuration

A configuration represents a set of build settings that define how your project
is built. A build configuration file is a plain-text file you use to specify the
build settings for a specific target or your entire project. Common configurations
include "Debug" and "Release," but you can create custom configurations to suit
your needs<sup>[*](https://developer.apple.com/documentation/xcode/adding-a-build-configuration-file-to-your-project)</sup>.

- **Usage:**
  - Configurations determine compiler flags, optimization settings, and other
  build-related parameters.
  - Each target in your Xcode project can have its own configuration settings.
  For example, you might have different optimization levels or preprocessor
  macros for Debug and Release builds.

- **Customization:** You can customize the build settings for each configuration
  in the Xcode project settings.

- **Common Scenarios:**
  - In a Debug configuration, you might include additional debug symbols and
  disable optimization for better debugging experience.
  - In a Release configuration, you might enable compiler optimizations and
  exclude unnecessary debugging information for a smaller app size.

In summary, a scheme is more about defining actions and behaviors, while a
configuration is about specifying how the project should be built with regard
to compiler flags, optimizations, and other build settings. Both work together
to provide flexibility and control over the building, running, and testing
processes in Xcode.

## DEBUG/DEVELOPMENT Configurations Name for SPM

Currently, when using Swift Package Manager packages in Xcode, SPM compiles
the package with reference to the name of the Build Configuration and
automatically selects whether to compile with debug or release, which
determines the compilation flag like DEBUG and this determines the architecture
of the final binary. This automatic selection may cause problems when using
a custom Build Configuration in Xcode other than the default “Debug” and “Release”.

Right now (October 2022) there is no particularly good way to map the Build
Configuration in Xcode to the SPM build environment <sup>[*](https://www.sobyte.net/post/2022-10/spm-in-xcode/#determination-based-on-build-configuration)</sup>.

## dSYM

Debug builds of an app place the debug symbols inside the compiled binary file
by default, while release builds of an app place the debug symbols in a
companion debug symbol (dSYM) file to reduce the size of the distributed app.
Each binary file in an app — the main app executable, frameworks, and app
extensions — has its own dSYM file. The compiled binary and its companion
dSYM file are tied together by a build UUID that's recorded by both the
built binary and dSYM file. If you build two binaries from the same source
code but with different Xcode versions or build settings, the build UUIDs
for the two binaries won't match. A binary and a dSYM file are only
compatible with each other when they have identical build UUIDs<sup>[*](https://developer.apple.com/documentation/xcode/building-your-app-to-include-debugging-information#Overview)</sup>.

Before building your app for distribution, verify that the [Debug Information Format](https://developer.apple.com/documentation/xcode/build-settings-reference#Debug-Information-Format)
build setting is set to DWARF with dSYM File. This generates the
necessary dSYM files, so you can diagnose crashes after releasing your app.

Starting with Xcode 14, bitcode is no longer required for watchOS and tvOS
applications, and the App Store no longer accepts bitcode submissions from
Xcode 14<sup>[*](https://developer.apple.com/documentation/xcode-release-notes/xcode-14-release-notes#Deprecations)</sup>.
Bitcode was an intermediate representation of a compiled program. It allowed
developers to submit apps to the App Store in this intermediate format rather
than a final binaries. This approach enabled Apple to optimize and recompile
app's code for different hardware architectures and future updates without
requiring developers to resubmit their apps<sup>[*](https://rocm.docs.amd.com/projects/llvm-project/en/latest/LLVM/llvm/html/BitCodeFormat.html)</sup>.

Since Bitcode is no longer part of the pipeline, dSYM files are directly
generated during the build process for all build configurations.
