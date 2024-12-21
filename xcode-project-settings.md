# Xcode Project Settings

- [Xcode Project Settings](#xcode-project-settings)
  - [iOS Keys for Info.plist](#ios-keys-for-infoplist)
  - [Build settings](#build-settings)
  - [Dependencies](#dependencies)
    - [Unused Dependencies](#unused-dependencies)
      - [Dead Code Elimination / Link-Time Optimization (LTO)](#dead-code-elimination--link-time-optimization-lto)
      - [App Store Submission and App Thinning](#app-store-submission-and-app-thinning)
      - [Dynamic Frameworks and Lazy Loading](#dynamic-frameworks-and-lazy-loading)

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
