---
layout: link
active_page: debugging-plugins-in-live
title: Debugging plugins in Live
---

# Debugging plugins in Live on macOS

## Introduction

**Is your plugin not showing up in Live when you try to debug it? Are you having trouble attaching a debugger or sanitizer?**

This article is a guide to help you debug plugins in Live on macOS. In modern macOS versions, Apple requires developers to ship applications with the [hardened runtime](https://developer.apple.com/documentation/security/hardened_runtime) enabled in order for them to be [notarized](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution?language=objc). This approach enhances security and user trust but can also complicate debugging for third-party plugin developers. This is especially true for Apple Silicon Macs, where the hardened runtime is enabled by default and security is more strict.

In applications such as Live that are code-signed with the hardened runtime, plugins that are not correctly [code-signed](code-signing-plugins) will not load and will not be visible in Live's browser. Furthermore the hardened runtime will, by default, prevent debuggers and sanitizers from attaching to or launching the Live application. There are a couple of different ways to work around this.

## Option 1: Disabling System Integrity Protection

The easiest way to work around the hardened runtime is to temporarily disable macOS System Integrity Protection (SIP). This can be done by booting your Mac into recovery mode and running the `csrutil disable` command. This will disable SIP, preventing the need for the plugin to be codesigned and allowing debuggers to attach to the Live application. For more information, see the [Apple documentation about Disabling and Enabling System Integrity Protection](https://developer.apple.com/documentation/security/disabling_and_enabling_system_integrity_protection). To check whether SIP is currently enabled or disabled, run the `csrutil status` command. It is recommended to re-enable SIP after you are done debugging.

## Option 2: Signing the Live application with the debugger entitlement

If you prefer to keep SIP enabled, you can code-sign the Live application yourself with the 'com.apple.security.get-task-allow' entitlement. This will allow you to attach a debugger. You can use the `codesign` command to re-sign the Live application. You can also sign with the `com.apple.security.cs.allow-dyld-environment-variables` entitlement to enable sanitizers. Note: These things are only suggested as a solution for debugging purposes, and should only be used for debugging purposes on your own machine - it is not advice that should be passed on to users.

Here is a bash script `codesign_live.sh` that will code-sign the Live application with an Ad-Hoc signature and the required entitlements for launching live with the debugger and/or sanitizer:

```bash
#!/bin/bash

# Exit on any error
set -e

# Path to Live application - modify if you need to code-sign a different version of Live
LIVE_APP="/Applications/Ableton Live 12 Suite.app"
TEMP_PLIST="/tmp/debug_entitlements.plist"

# Extract current entitlements
codesign -d --entitlements - "$LIVE_APP" --xml > $TEMP_PLIST

# Add required debugging entitlements
# - com.apple.security.get-task-allow: Enables debugger attachment to the application
# - com.apple.security.cs.allow-dyld-environment-variables: Allows sanitizers

# Add other entitlements, which may be useful for debugging
# - com.apple.security.cs.disable-library-validation: Allows loading of unsigned libraries/plugins
# - com.apple.security.cs.allow-unsigned-executable-memory: Permits JIT compilation and dynamic code generation

for entitlement in \
    com.apple.security.cs.disable-library-validation \
    com.apple.security.cs.allow-unsigned-executable-memory \
    com.apple.security.get-task-allow \
    com.apple.security.cs.allow-dyld-environment-variables
do
    /usr/libexec/PlistBuddy -c "Add :$entitlement bool true" $TEMP_PLIST 2>/dev/null || true
done

# Re-sign the application
codesign --force --options runtime --sign - --entitlements $TEMP_PLIST "$LIVE_APP"

# Cleanup
rm $TEMP_PLIST

```

In order to use this script, you'll need to make it executable and run it.

```bash
chmod +x codesign_live.sh
./codesign_live.sh

```


Note: This bash script has been adapted from the script provided by github user talaviram [here](https://gist.github.com/talaviram/1f21e141a137744c89e81b58f73e23c3).

If you do code-sign the Live application, you will need to re-sign it every time you update Live. You should also make sure that you test your plugin in an unmodified version of Live if you want to be check how it will behave on a user's machine.

## Debugging AUv3 plugins on Apple Silicon Macs

AUv3 plugins can be loaded in-process or out-of-process depending on the way they are built, which changes the way that you need to attach a debugger. Apple has provide a guide about debugging out-of-process AUv3 plugins [here](https://developer.apple.com/documentation/audiotoolbox/debugging-out-of-process-audio-units-on-apple-silicon?language=objc)

***

[Back](index)
