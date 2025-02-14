# Xcode and simulator

- [Xcode and simulator](#xcode-and-simulator)
  - [Home Directory](#home-directory)
  - [Code Snippets](#code-snippets)
  - [Provisioning Profiles](#provisioning-profiles)
  - [Running custom scripts during a build](#running-custom-scripts-during-a-build)

## Home Directory

It is possible to store files such as logs on device/simulator. In order to
access the path, when the app is hooked to Xcode, can pause or place a
breakpoint in Xcode and run the following command in the LLDB console:

```Swift
po NSHomeDirectory()
```

You can get a list of supported preview device names by using the `xcrun`
command in the Terminal app:

```bash
xcrun simctl list devicetypes
```

To list all the frameworks, including the private ones, in iOS:

```bash
ls \"$(xcode-select -p)/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/System/Library/Frameworks\"
```

## Code Snippets

Code snippet is a reusable piece of code or code template that you can save and
quickly insert into your source code.

Code snippets are stored in the following path:

```bash
~/Library/[username]/Developer/Xcode/UserData/CodeSnippets/
```

## Provisioning Profiles

Xcode stores all the provisioning profiles in the following directory:

```bash
~/Library/Developer/Xcode/UserData/Provisioning\ Profiles
```

## Running custom scripts during a build

Xcode uses the following format for displaying warning/error/note[<sup>*</sup>](https://developer.apple.com/documentation/xcode/running-custom-scripts-during-a-build):

```text
[filename]:[linenumber]: error | warning | note : [message]
```
