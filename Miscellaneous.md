# Miscellaneous

- [Miscellaneous](#miscellaneous)
  - [Download Test Files](#download-test-files)
  - [iOS Release Versions and Build Numbers](#ios-release-versions-and-build-numbers)
  - [Underscored Attributes Reference](#underscored-attributes-reference)
  - [Errors](#errors)
  - [Date Formats](#date-formats)
  - [Content Delivery Network (CDN)](#content-delivery-network-cdn)
  - [Logging](#logging)
  - [Code Review](#code-review)

## Download Test Files

In case we want to hit an endpoint to download a large file to test, for
instance, a progress view or task cancellation, [this website](http://xcal1.vodafone.co.uk)
offers a few endpoints.

The only catch is that the endpoint isn't using SSL pinning; therefore, the
project needs to add [NSAppTransportSecurity](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity/).

## iOS Release Versions and Build Numbers

The [Beta Wiki](https://betawiki.net/wiki/Main_Page) website is a good reference
to find details about iOS releases.

## Underscored Attributes Reference

There are two kinds of attributes in Swift — those that apply to declarations
and those that apply to types. An attribute provides additional information
about the declaration or type. For example, the `discardableResult` attribute
on a function declaration indicates that, although the function returns a value,
the compiler shouldn’t generate a warning if the return value is unused.

There's two lists:

1. [Unofficial/Undocumented Attributes](https://github.com/swiftlang/swift/blob/main/docs/ReferenceGuides/UnderscoredAttributes.md)
2. [Official/Documented Attributes](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/attributes/)

## Errors

The [OSStatus](https://www.osstatus.com) website is a good place to map an error
code to some Apple Error.

## Date Formats

The format string uses [the format patterns](http://unicode.org/reports/tr35/tr35-dates.html#Date_Format_Patterns)
from the Unicode Technical Standard #35. The symbols are tabulated in the
[Date Field Symbol table](http://unicode.org/reports/tr35/tr35-dates.html#Date_Field_Symbol_Table).

A common mistake is to use YYYY. yyyy specifies the calendar year whereas YYYY
specifies the year (of “Week of Year”), used in the ISO year-week calendar.
In most cases, yyyy and YYYY yield the same number, however they may be different.
Typically you should use the calendar year.

## Content Delivery Network (CDN)

A CDN is a network of geographically dispersed servers used to deliver static content.
CDN servers cache static content like images, videos, CSS, JavaScript files, etc.

## Logging

Note that some of the `OSLogType` are actually equivalent to another one.
This includes the icon and also the tooltip that shows up when hovering over:

- `notice` is just like `default`
- `error` is just like `warning`
- `fault` is just like `critical`

## Code Review

> In general, reviewers should favor approving a diff once it is in a state
> where it definitely improves the overall code health of the system being
> worked on, even if the diff isn’t perfect”.
>
> _(Google Engineering Practices documentation)_

The purpose of the review is to keep the quality of the codebase at
or above current levels across reliability, functionality, and
maintainability. The purpose is not to create un-constructive
criticism or start philosophical engineering discussions.

As a developer, create small diffs. Your reviewer will need less context
and review it for you faster. No one ever said this engineer must be very
smart because his/her diffs are always huge.

Be courteous and respectful to your fellow colleagues. Your comments should
be about the code and not the developer who wrote it. Use the appropriate word choices.
