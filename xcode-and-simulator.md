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

It's possible to increase custom script run phase by introducing INPUT and
OUTPUT files. Here’s the low down on the impact of specifying input and output
files with script phases in Xcode[<sup>*</sup>](https://indiestack.com/2014/12/speeding-up-custom-script-phases/):

- Lack of modification to any of the listed input files encourages Xcode not
to run the script phase. Hooray! Fastness!
- Non-existence of any listed output file compels Xcode to run your script phase.
- The number of input and output file paths is passed to the script
as `${SCRIPT_INPUT_FILE_COUNT}` and `${SCRIPT_OUTPUT_FILE_COUNT}` environment variables.
- Each input and output path is passed as e.g. `${SCRIPT_INPUT_FILE_0}`,
`${SCRIPT_OUTPUT_FILE_1}`, etc.

In practice, what does this mean when you go looking to speed up your
script phase? It means you should:

- List as an input every file or folder that may affect the results of your
script phase.
- List at least one output, even if your script doesn’t leave any reliable artifacts.

Adding a bogus file to the derived files folder, as the output can indicate that
the script has run. It has the effect of causing a “clean” of the project to wipe
out the artificial file and thus cause the script phase to run again even
though none of the input files may have changed.
