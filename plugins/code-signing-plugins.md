---
layout: link
active_page: code-signing-plugins
title: Code-signing plugins for Live on macOS
---

# Code-signing plugins for Live on macOS

## Introduction

Since Live 10.1.4 plugins should be properly code-signed in order to be loaded in Live on macOS. This is a requirement of the macOS [hardened runtime](https://developer.apple.com/documentation/security/hardened_runtime), which was enabled in Live 10.1.4. If you want your VST2, VST3 or Audio Unit plugins to be loaded in Live, you will need to code-sign them correctly. If an invalid signature is detected, Live will not load the plugin.

## Code-signing plugins
If you want to just run your plugin in Live on your own machine, you can use the "Ad-Hoc" code-signing method. This is the easiest method, but it will not work for distributing your plugin to other users - it is only intended for debugging purposes on your own machine. To code-sign your plugin for distribution you will need to [Join the Apple Developer Program](https://developer.apple.com).

For more information about code-signing, see the [Apple documentation on the topic](https://developer.apple.com/documentation/xcode/creating-distribution-signed-code-for-the-mac).


<!-- 
### Ad-Hoc code-signing

To code-sign your plugin with an Ad-Hoc signature you don't require an Apple Developer ID. You can then use the `codesign` command on the command line to sign the plugin, or you can configure code-signing in e.g. XCode's build settings or CMake, depending on your build system.

```bash
codesign --force --sign /path/to/your/plugin.vst3

```

You can verify the signature by running:

```bash
codesign --verify --verbose=4  /path/to/your/plugin.vst3

```

### Code-signing for distribution

To code-sign your plugin for distribution you will need to [Join the Apple Developer Program](https://developer.apple.com).

In order to find the correct name for the code-signing identity, you can run:

```bash
security find-identity -p codesigning -v

```

Then replace `Your Name` and `Your Developer ID` with the correct values for your code-signing identity in the following command:


```bash
codesign --force --sign "Apple Development: Your Name (Your Developer ID)" /path/to/your/plugin.vst3

```

You can verify the signature by running:

```bash
codesign --verify --verbose=4  /path/to/your/plugin.vst3

``` -->

***

[Back](index)
