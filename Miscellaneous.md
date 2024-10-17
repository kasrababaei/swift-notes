# Miscellaneous

## Download Test Files

In case we want to hit an endpoint to downlooad a large file to test, for instance, a progress view or task cancellation, [this website](http://xcal1.vodafone.co.uk) offers a few endpoints.

The only catch is that the endpoint isn't using SSL pinning; therefore, the project needs to add [NSAppTransportSecurity](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity/).

## iOS Release Versions and Build Numbers

The [Beta Wiki](https://betawiki.net/wiki/Main_Page) website is a good reference to find details about iOS releases.

## Underscored Attributes Reference

There are two kinds of attributes in Swift — those that apply to declarations and those that apply to types. An attribute provides additional information about the declaration or type. For example, the discardableResult attribute on a function declaration indicates that, although the function returns a value, the compiler shouldn’t generate a warning if the return value is unused.

There's two lists:

1. [Unofficial/Undocumented Attributes](https://github.com/swiftlang/swift/blob/main/docs/ReferenceGuides/UnderscoredAttributes.md)
2. [Official/Documented Attributes](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/attributes/)

## Errors

[Apple API Errors](https://www.osstatus.com)