---
layout: linkkit
active_page: linkkit
title: LinkKit
---

# LinkKit

iOS SDK for [Ableton Link](https://ableton.com/link), a technology that synchronizes
musical beat, tempo, phase, and start/stop commands across multiple applications running on
one or more devices. Applications on devices connected to a local network discover each
other automatically and form a musical session in which each participant can perform
independently: anyone can start or stop while still staying in time. Anyone can change
the tempo, the others will follow. Anyone can join or leave without disrupting the
session.

This site contains documentation and reference material for the [LinkKit SDK](https://github.com/Ableton/LinkKit).

We strongly recommend reading all of the content below, but please pay special attention
to the [user interface guidelines](#user-interface-guidelines) and the [test
plan](/link/#test-plan) in order to make sure that your app is consistent with others in the
Link ecosystem. If you haven't read the conceptual overview on the [Link](/link) page,
please start with that.

### License

Usage of LinkKit is governed by the [Ableton Link SDK license](https://github.com/Ableton/LinkKit/blob/master/LICENSE.md).

## Integration Guide

The LinkKit SDK is distributed as a zip file attached to a release in the LinkKit
[repo](https://github.com/Ableton/LinkKit). You can find the latest release on the
[releases tab](https://github.com/Ableton/LinkKit/releases). Apps **should** be built
against an official release for final submission to the App Store. Official releases are
those not marked "Pre-release."  
In case of questions please open a GitHub issue or contact [link-devs@ableton.com](mailto:link-devs@ableton.com).

### Getting Started

Download the `LinkKit.zip` file attached to the latest release. A `LinkKit.zip` file has
the following contents:

- `LinkKit.xcframework`: A bundle containing the static librares for all supported platforms and the corresponding headers:
  - [`ABLLink.h`](https://github.com/Ableton/LinkKit/blob/master/LinkKit/ABLLink.h):
  Pure C header containing the Link API.
  - [`ABLLinkSettingsViewController.h`](https://github.com/Ableton/LinkKit/blob/master/LinkKit/ABLLinkSettingsViewController.h):
  Objective-C header containing `UIViewController` subclass that is used to display the
  Link preference pane.
- `LinkKitResources.bundle`: A bundle containing localization strings.
- [User interface assets](https://github.com/Ableton/LinkKit/tree/master/assets)
- [LinkHut](https://github.com/Ableton/LinkKit/tree/master/examples/LinkHut): Very simple
  app to be used as example code and for testing integrations. It should build and run
  in-place without modification.

In order to build and link against LinkKit add `LinkKit.xcframework` to your linker dependencies.
LinkKit is implemented in C++, so you may also need to add `-lc++` to your link line if you're not already using C++ in your project. This is needed to pull in the C++ standard library.

In case your app supports localization, add `LinkKitResources.bundle` to "Copy Bundle Resources" in the "Build Phases" section of the target settings in Xcode.

### iOS 14+ Compatibility
Link sends multicast UDP messages to 224.76.78.75:20808 on the local area network to communicate with other peers. With the release of iOS 14 Apple added security measures that require user consent and special entitlements for an app to send and receive multicast network messages.

When your app launches or the user activates Link for the first time, the user will be asked to give the application permission to "find and connect to devices on the local network". If the user does not allow this, Link will not be able to connect to other peers.
Therefore, you must add an [NSLocalNetworkUsageDescription](https://developer.apple.com/documentation/bundleresources/information_property_list/nslocalnetworkusagedescription) entry to `Info.plist`. This can also be used to give the user more detailed information on why network access is necessary.

When using Xcode 12 or later, and when building for iOS 14 and above, you will have to request the `com.apple.developer.networking.multicast` entitlement from Apple to run your app with Link on a device. This is even required running a build directly from Xcode. When requesting the entitlement from Apple, you can reference this documentation and the [Ableton Link product page](https://www.ableton.com/link/).

Documentation and further information from Apple can be found [here](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_networking_multicast) and [here](https://developer.apple.com/news/?id=0oi77447).

### User Interface Guidelines

LinkKit includes a Link preference pane that must be added to an app's user interface.
The appearance and behavior of the preference pane itself is not configurable, but you
must make the choice of where and how to expose access to the preference pane within the
app. In order to provide a consistent user experience across all Link-enabled apps, we
have developed [UI integration guidelines (PDF)](https://github.com/Ableton/LinkKit/blob/master/docs/Ableton_Link_UI_Guidelines.pdf)
that provide guidance on this matter. Please follow them carefully.

Also included in this repo are
[assets](https://github.com/Ableton/LinkKit/tree/master/assets) to be used if you choose
to put a Link button in your app. All assets relating to the Ableton Link identity will
be provided by Ableton and all buttons, copy, and labels should follow the [UI
integration guidelines (PDF)](https://github.com/Ableton/LinkKit/blob/master/docs/Ableton_Link_UI_Guidelines.pdf).

### Host Time

Host time as used in the API always refers to the system host time and is the same
coordinate system as values returned by
[`mach_absolute_time`](https://developer.apple.com/library/mac/qa/qa1398/_index.html) and
the `mHostTime` field of the `AudioTimeStamp` structure.

All host time values used in the Link API refer to host times at output. This is the
important value for the library to know, since it must coordinate the timelines of
multiple devices such that audio for the same beat time is hitting the output of those
devices at the same moment. This is made more complicated by the fact that different
devices (and even the same device in different configurations) can have different output
latencies.

In the audio callback, the system provides an `AudioTimeStamp` value for the audio
buffer. The `mHostTime` field of this structure represents the host time at which the
audio buffer will be passed to the hardware for output. Adding the output latency (see
`AVAudioSession.outputLatency`) to this value will result in the correct host time at
output for the *beginning* of that buffer. To get the host time at output for the end of
the buffer, you would just add the buffer duration. For an example of this calculation,
see the [LinkHut example
project](https://github.com/Ableton/LinkKit/blob/master/examples/LinkHut/LinkHut/AudioEngi
ne.m).

Note that if your app adds additional software latency, you will need to add this as well
in the calculation of the host time at output. Also note that the
`AVAudioSession.outputLatency` property can change, so you should update your output
latency in response to the
[`AVAudioSessionRouteChangedNotification`](https://developer.apple.com/library/ios/documen
tation/AVFoundation/Reference/AVAudioSession_ClassReference/#//apple_ref/c/data/AVAudioSes
sionRouteChangeNotification) in order to maintain the correct values in your latency
calculations.

### Link API Usage

This section contains extended discussion on the contents of the C header
[ABLLink.h](https://github.com/Ableton/LinkKit/blob/master/LinkKit/ABLLink.h) and the
Objective-C header [ABLLinkSettingsViewController.h](https://github.com/Ableton/LinkKit/blob
/master/LinkKit/ABLLinkSettingsViewController.h), which together make up the Link API.

#### Initialization and Destruction

An ABLLink library instance is created with the `ABLLinkNew` function. Creating a library
instance is a pre-requisite to using the rest of the API. It
is recommended that the library instance be created on the main thread during app
initialization and that it be preserved for the lifetime of the app. There should not be
a reason to create and destroy multiple instances of the library during an app's
lifetime. To cleanup the instance on app shutdown, call `ABLLinkDelete`.

An app must provide an initial tempo when creating an instance of the library. The tempo
is required because a library instance is initialized with a new timeline that starts
running from beat 0. The initial tempo provided to `ABLLinkNew` determines the rate of
progression of this beat timeline until the app sets a new tempo or a new tempo comes in
from the network. It is important that a valid tempo be provided to the library at
initialization time, even if it's just a default value like 120bpm.

#### Active, Enabled, and Connected
Once an ABLLink instance is created, in order for it to start attempting to connect to
other participants on the network, it must be both *active* and *enabled*. The active and
enabled properties are two independent boolean conditions, the first of which is
controlled by the app, and the second by the end user. So Link needs permission from both
the app and the end user before it starts communicating on the network.

The enabled state is controlled directly by the user via the
[`ABLLinkSettingsViewController.h`](https://github.com/Ableton/LinkKit/blob/master/LinkKit
/ABLLinkSettingsViewController.h). It
persists across app runs, so if the user enables Link they don't have to re-enable it
every time they re-launch the app. Since it is a persistent value that is visible and
meaningful to the user, the API does not allow it to be modified programmatically by the
app. However, the app can observe the enabled state via the
`ABLLinkIsEnabled` function and the
`ABLLinkSetIsEnabledCallback` callback
registration function. These should only be needed to update UI elements that reflect the
Link-enabled state. If you are depending on the enabled state in audio code, you're doing
something wrong (you should probably be using
`ABLLinkIsConnected` instead. More on that soon...)

The active state is controlled by the app via the
`ABLLinkSetActive` function. This is primarily used to
implement [background behavior](#background-behavior) - by calling
`ABLLinkSetActive(false)` when going to the background, an app can make sure that the
ABLLink instance is not communicating on the network when it's not needed or expected.

When an ABLLink instance is both active and enabled, it will attempt to find other
participants on the network in order to form a Link session. When at least one other
participant has been found and a session has been formed, then the instance is considered
connected. This state can be queried with the
`ABLLinkIsConnected` function.

Start Stop Sync is an opt in feature. To allow the user to enable Start Stop Sync with
a toggle in the ABLLinkSettingsViewController a Boolean entry `YES` with the key
`ABLLinkStartStopSyncSupported` must be added to `Info.plist`.
The app can observe the state via the
`ABLLinkIsStartStopSyncEnabled` function
and the
`ABLLinkSetIsStartStopSyncEnabledCallback` callback registration function. These should
only be needed to update UI elements that
reflect the Start Stop Sync enabled state. The interface to the start/stop state behaves
the same wether Start Stop Sync is enabled or not. The only difference is that changes are
not kept in sync with other peers. This way the app does not have to change its behavior
depending on the feature being enabled or disabled.

Link Audio is an opt in feature that provides the ability to send audio channels to peers
in a Link session. To allow the user to enable audio sharing a Boolean entry `YES` with
the key `ABLLinkAudioSupported` must be added to the `Info.plist`. A default peer name
that identifies the app to other peers in the Link session can be set with a String entry
with the key `ABLLinkPeerName`. This name can be changed by the user in the settings view
at any time.

### App Life Cycle

In order to provide the best user experience across the ecosystem of Link-enabled apps,
it's important that apps take a consistent approach towards Link with regards to life
cycle management. Furthermore, since the Link library does not have access to all of the
necessary information to correctly respond to life cycle events, app developers must
follow the life cycle guidelines below in order to meet user expectations. Please
consider these carefully.

- When an app moves to the background and it is known that no audio will be generated by
  the app while in the background, Link should be deactivated by calling
  `ABLLinkSetActive(false)` - *even if background audio has been enabled for the app*.
  This helps prevent the confusing situation where a silent background app discovers and
  connects to other Link enabled apps and devices.
- There are situations where an app has to start playback while in the background, such as when
  Start Stop Sync is enabled, if it is part of an Audiobus or IAA session or if is
  listening to MIDI input. In these cases, Link should remain active when the app moves to
  the background.
- When an app is active, Link should be active. If an app deactivates Link using the
  `ABLLinkSetActive` function when going to the
  background, it must re-activate Link when becoming active again. For this reason, it is
  recommended that a call to `ABLLinkSetActive(true)` be included in the
  `applicationDidBecomeActive` method of the application delegate. Calling this function
  when Link is already active causes no harm, so this can be included unconditionally.
- It is possible for an app to be added to an Audiobus or IAA session while it is in the
  background. If Link was deactivated when moving to the background and the app is then
  added to an Audiobus or IAA session, Link should be re-activated in anticipation of
  playing audio as part of the session. Conversely, if an app in the background is
  ejected from an Audiobus or IAA session while not playing, and therefore no longer has
  the possibility to generate audio, it should deactivate Link. Handling these cases
  correctly will generally require listening to the
  [ABConnectionsChangedNotification](https://developer.audiob.us/doc/_a_b_audiobus_control
  ler_8h.html#a336d7bc67873e51abf5b09f7fe15b9f4).

Please see the LinkHut
[AppDelegate.m](https://github.com/Ableton/LinkKit/blob/master/examples/LinkHut/LinkHut/Ap
pDelegate.m) file for a basic example of implementing the app life cycle guidelines,
although it does not support Audiobus or IAA.

### Audiobus

We have worked closely with the developers of Audiobus to provide some additional
features when using Link-enabled apps within Audiobus. In order to take advantage of
these additional features, please be sure to build against the latest available version
of the Audiobus SDK when adding Link to your app. No code changes are required on your
part to enable the Audiobus-Link integration, but please be sure to check the
"Link-enabled" box in your Audiobus profile so that your app will be listed correctly in
the Audibous app directory.

### Other Sync Technologies

We recommend making Link mutually exclusive with other sync technologies that may be
supported by your app, such as MIDI Clock or WIST. Having two concurrent clock sources
fighting each other will degrade the Link session and compromise the user experience. The
`ABLLinkSetIsEnabledCallback` callback registration function can be used to observe when
Link has been enabled by the user in order to disable UI elements and functionality of
other sync technologies.

## Test Plan

The [Ableton Link test plan](/link/#test-plan) covers the shared Link behavior and
includes additional iOS-specific checks for LinkKit apps.

## Promoting Link Integration

After investing the time and effort to add Link to your app, you will probably want to
tell the world about it. When you do so, please be sure to follow our [Ableton Link
promotion guidelines (PDF)](https://github.com/Ableton/LinkKit/blob/master/docs/Ableton_Link_Promotion.pdf).
The Link badge referred to in the guidelines can be found in the
[assets](https://github.com/Ableton/LinkKit/tree/master/assets) folder. You can also find
additional info and images in our [press kits](https://ableton.com/press) and use them as
you please.
