# Xcode and simulator

It is possible to store files such as logs on device/simulator. In order to access the path, when the app is hooked to Xcode, can pause or place a breakpoint in Xcode and run the following command in the LLDB console:

```Swift
po NSHomeDirectory()
```

You can get a list of supported preview device names by using the xcrun command in the Terminal app:

```text
xcrun simctl list devicetypes
```
