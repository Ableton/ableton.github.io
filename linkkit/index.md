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
plan](#test-plan) in order to make sure that your app is consistent with others in the
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

- `libABLLink.a`: A static library containing the implementation of Link. This file is
  **not** in the repo - you must download a release to get it.
- [`ABLLink.h`](api-reference/#abllinkh): Pure C header containing the Link API.
- [`ABLLinkSettingsViewController.h`](api-reference/#abllinksettingsviewcontrollerh):
  Objective-C header containing `UIViewController` subclass that is used to display the
  Link preference pane.
- [User interface assets](https://github.com/Ableton/LinkKit/tree/master/assets)
- [LinkHut](https://github.com/Ableton/LinkKit/tree/master/examples/LinkHut): Very simple
  app to be used as example code and for testing integrations. It should build and run
  in-place without modification.

In order to build and link against `libABLLink.a`, make sure that the location of the
header files is added to the include path of your project and location of the library
added to the linker path. `libABLLink.a` is implemented in C++, so you may also need to
add `-lc++` to your link line if you're not already using C++ in your project. This is
needed to pull in the C++ standard library.

### User Interface Guidelines

LinkKit includes a Link preference pane that must be added to an app's user interface.
The appearance and behavior of the preference pane itself is not configurable, but you
must make the choice of where and how to expose access to the preference pane within the
app. In order to provide a consistent user experience across all Link-enabled apps, we
have developed [UI integration guidelines (PDF)](downloads/Ableton Link UI Guidelines.pdf)
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
[ABLLink.h](api-reference/#abllinkh) and the Objective-C header
[ABLLinkSettingsViewController.h](api-reference/#abllinksettingsviewcontrollerh), which
together make up the Link API.

#### Initialization and Destruction

An ABLLink library instance is created with the [`ABLLinkNew`](api-reference/#abllinknew)
function. Creating a library instance is a pre-requisite to using the rest of the API. It
is recommended that the library instance be created on the main thread during app
initialization and that it be preserved for the lifetime of the app. There should not be
a reason to create and destroy multiple instances of the library during an app's
lifetime. To cleanup the instance on app shutdown, call
[`ABLLinkDelete`](api-reference/#abllinkdelete).

An app must provide an initial tempo when creating an instance of the library. The tempo
is required because a library instance is initialized with a new timeline that starts
running from beat 0. The initial tempo provided to
[`ABLLinkNew`](api-reference/#abllinknew) determines the rate of progression of this beat
timeline until the app sets a new tempo or a new tempo comes in from the network. It is
important that a valid tempo be provided to the library at initialization time, even if
it's just a default value like 120bpm.

#### Active, Enabled, and Connected
Once an ABLLink instance is created, in order for it to start attempting to connect to
other participants on the network, it must be both *active* and *enabled*. The active and
enabled properties are two independent boolean conditions, the first of which is
controlled by the app, and the second by the end user. So Link needs permission from both
the app and the end user before it starts communicating on the network.

The enabled state is controlled directly by the user via the
[`ABLLinkSettingsViewController.h`](api-reference/#abllinksettingsviewcontrollerh). It
persists across app runs, so if the user enables Link they don't have to re-enable it
every time they re-launch the app. Since it is a persistent value that is visible and
meaningful to the user, the API does not allow it to be modified programmatically by the
app. However, the app can observe the enabled state via the
[`ABLLinkIsEnabled`](api-reference/#abllinkisenabled) function and the
[`ABLLinkSetIsEnabledCallback`](api-reference/#abllinksetisenabledcallback) callback
registration function. These should only be needed to update UI elements that reflect the
Link-enabled state. If you are depending on the enabled state in audio code, you're doing
something wrong (you should probably be using
[`ABLLinkIsConnected`](api-reference/#abllinkisconnected) instead. More on that soon...)

The active state is controlled by the app via the
[`ABLLinkSetActive`](api-reference/#abllinksetactive) function. This is primarily used to
implement [background behavior](#background-behavior) - by calling
`ABLLinkSetActive(false)` when going to the background, an app can make sure that the
ABLLink instance is not communicating on the network when it's not needed or expected.

When an ABLLink instance is both active and enabled, it will attempt to find other
participants on the network in order to form a Link session. When at least one other
participant has been found and a session has been formed, then the instance is considered
connected. This state can be queried with the
[`ABLLinkIsConnected`](api-reference/#abllinkisconnected) function.

Start Stop Sync is an opt in feature. To allow the user to enable Start Stop Sync with
a toggle in the ABLLinkSettingsViewController a Boolean entry `YES` under the key
`ABLLinkStartStopSyncSupported` must be added to `Info.plist`.
The app can observe the state via the
[`ABLLinkIsStartStopSyncEnabled`](api-reference/#abllinkisstartstopsyncenabled) function
and the
[`ABLLinkSetIsStartStopSyncEnabledCallback`](api-reference/#abllinksetisstartstopsyncenabledenabledcallback)
callback registration function. These should only be needed to update UI elements that
reflect the Start Stop Sync enabled state. The interface to the start/stop state behaves
the same wether Start Stop Sync is enabled or not. The only difference is that changes are
not kept in sync with other peers. This way the app does not have to change its behavior
depending on the feature being enabled or disabled.

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
  [`ABLLinkSetActive`](api-reference/#abllinksetactive) function when going to the
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

Below are a set of user interactions that are expected to work consistently across all
Link-enabled apps. In order to provide the best user experience, it's important that apps
behave consistently with respect to these test cases. *Please verify that your app passes
__all__ of the test cases before submitting to the App Store.* Apps that do not pass this
test suite will not be considered conforming Link integrations.

### Tempo Changes

#### TEMPO-1: Tempo changes should be transmitted between connected apps.

- Open LinkHut, press **Play**, and set Link to **Enabled**.
- Open App and set Link to **Enabled**
- Without starting to play, change tempo in App **&rArr;** LinkHut clicks should speed up
  or slow down to match the tempo specified in the App.
- Start playing in the App **&rArr;** App and LinkHut should be in sync
- Change tempo in App and in LinkHut **&rArr;** App and LinkHut should remain in sync

#### TEMPO-2: Opening an app with Link enabled should not change the tempo of an existing Link session.

- Open App and set Link to **Enabled**.
- Set App tempo to 100bpm.
- Terminate App.
- Open LinkHut, press **Play** and set Link to **Enabled**.
- Set LinkHut tempo to 130bpm.
- Open App **&rArr;** Link should be connected (“1 Link”) and the App and LinkHut’s tempo
  should both be 130bpm.

#### TEMPO-3: When connected, loading a new document should not change the Link session tempo.

- Open LinkHut, press **Play**, and set Link to **Enabled**.
- Set LinkHut tempo to 130bpm.
- Open App and set Link to **Enabled** **&rArr;** LinkHut’s tempo should not change.
- Load new Song/Set/Session with a tempo other than 130bpm **&rArr;** App and LinkHut
  tempo should both be 130bpm.

#### TEMPO-4: Tempo range handling.

- Open LinkHut, press **Play**, and set Link to **Enabled**.
- Open App, start Audio, and set Link to **Enabled**.
- Change tempo in LinkHut to **20bpm** **&rArr;** App and LinkHut should stay in sync.
- Change Tempo in LinkHut to **999bpm** **&rArr;** App and LinkHut should stay in sync.
- If App does not support the full range of tempos supported by Link, it should stay in
  sync by switching to a multiple of the Link session tempo.

#### TEMPO-5: Enabling Link does not change app's tempo if there is no Link session to join.
- Open App, start playing.
- Change App tempo to something other than the default.
- Set Link to **Enabled** **&rArr;** App's tempo should not change.
- Change App tempo to a new value (not the default).
- Set Link to **Disabled** **&rArr;** App's tempo should not change.

### Beat Time

These cases verify the continuity of beat time across Link operations.

#### BEATTIME-1: Enabling Link does not change app's beat time if there is no Link session to join.
- Open App, start playing.
- Set Link to **Enabled** **&rArr;** No beat time jump or audible discontinuity should
  occur.
- Set Link to **Disabled** **&rArr;** No beat time jump or audible discontinuity should
  occur.

#### BEATTIME-2: App's beat time does not change if another participant joins its session.
- Open App and set Link to **Enabled**.
- Start playing.
- Open LinkHut and set Link to **Enabled** **&rArr;** No beat time jump or audible
  discontinuity should occur in the App.

**Note**: When joining an existing Link session, an app should adjust to the existing
session's tempo and phase, which will usually result in a beat time jump. Apps that are
already in a session should never have any kind of beat time or audio discontinuity when
a new participant joins the session.

### Start Stop States

#### STARTSTOPSTATE-1: Listening to start/stop commands from other peers.
- Open App, set Link and Start Stop Sync to **Enabled**.
- Open LinkHut, set Link and Start Stop Sync to **Enabled** and press **Play** **&rArr;**
App should start playing according to its quantization.
- Stop playback in LinkHut **&rArr;** App should stop playing.

#### STARTSTOPSTATE-2: Sending start/stop commands to other peers.
- Open LinkHut, set Link and Start Stop Sync to **Enabled** and press **Play**.
- Open App, set Link and Start Stop Sync to **Enabled** **&rArr;** App should not be
playing while LinkHut continues playing.
- Start playback in App **&rArr;** App should join playing according to its quantization.
- Stop playback in App **&rArr;** App and LinkHut should stop playing.
- Start playback in App **&rArr;** App and LinkHut should start playing according to
their quantizations.

### Audio Engine

These cases verify the correct implementation of latency compensation within an app's
audio engine.

#### AUDIOENGINE-1: Correct alignment of app audio with shared session

- Connect iOS Device to the audio input of a computer or alternatively use Inter-Device
  Audio on OSX 10.11.
- Open LinkHut, press **Play**, and set Link to **Enabled**.
- Open App and set Link to **Enabled**.
- Start playing audio (preferably a short, click-like sample) with notes on the same
  beats as LinkHut.
- Record incoming audio on desktop within application of choice.
- Validate whether onset of the sample aligns with the pulse generated by LinkHut
  (tolerance: less than 3 ms).

#### AUDIOENGINE-2: Latency is updated when AVAudioSession route changes

- Make sure audio is played back through the internal speakers of the iOS device.
- Open LinkHut, press **Play**, and set Link to **Enabled**.
- Open App and set Link to **Enabled**.
- Start playing audio (preferably a short, click-like sample) with notes on the same
  beats as LinkHut.
- Connect iOS Device to the audio input of a computer or switch on Inter-Device Audio on
  OSX 10.11.
- Record incoming audio on desktop within application of choice.
- Validate whether onset of the sample aligns with the pulse generated by LinkHut
  (tolerance less than 3 ms).

**Note**: When a cable is connected to the headphone-jack of an iOS device or
Inter-Device Audio is turned on during operation, latency may change and must be
accounted for. See [host time at output](#host-time-at-output).

### Background Behavior

These cases test the correct implementation of the [app life cycle
guidelines](#app-life-cycle).

#### BACKGROUND-1: Link remains active when going to the background while playing audio.

- Open LinkHut, press **Play**, and set Link to **Enabled**.
- Open App and set Link to **Enabled**.
- Start playing audio.
- Open LinkHut, press **Settings** **&rArr;** there should be 1 connected app

#### BACKGROUND-2: Link is deactivated when going to the background and audio will not be played while in background.

- Open LinkHut, press **Play**, and set Link to **Enabled**.
- Open App and set Link to **Enabled**.
- Stop App from playing audio and put it in the background
- Open LinkHut, press **Settings** **&rArr;** there should be 0 connected apps.
- Bring App to the foreground again **&rArr;** there should be a notification “1 Link”
  and the Link settings should reflect this.
- Disable and enable Link in App **&rArr;** there should be a notification “1 Link” and
  the Link settings should reflect this.

**Note:** This is the expected behavior even if the App's background audio mode is
enabled. Whenever the App goes to the background and it's known that the App will not be
playing audio or processing MIDI while in the background (not receiving MIDI, not
connected to IAA or Audiobus), Link should be deactivated.

#### BACKGROUND-3: Link remains active when going to background while Start Stop Sync is enabled (if supported).

- Open LinkHut and set Link to **Enabled** and set Sync Start/Stop to **Enabled**.
- Switch to the App and set Link to **Enabled** and set Sync Start/Stop to **Enabled**
  **&rArr;** there should be a notification "1 Link" and the Link settings should reflect this.
- Make sure the App is not playing and switch to LinkHut **&rArr;** No notification is
  presented. The Link settings should still indicate 1 connected App.
- Start and stop playback in LinkHut **&rArr;** App should start and stop playback accordingly.

**Note**: While Start Stop Sync is enabled Link must remain active even while not playing
in the background because the App must be prepared to start playing at anytime.


#### BACKGROUND-4: Link remains active when going to background while part of an IAA or Audiobus session (if supported).

- Open LinkHut, press **Play**, and set Link to **Enabled**.
- Open Audiobus and add the App as **Input**.
- Switch to the App and set Link to **Enabled** **&rArr;** there should be a notification
  "1 Link" and the Link settings should reflect this.
- Make sure the App is not playing and switch to LinkHut **&rArr;** No notification is
  presented. The Link settings should still indicate 1 connected App.

**Note**: While connected to Audiobus/IAA Link must remain active even while not playing
in the background because the App must be prepared to start playing at anytime.

#### BACKGROUND-5: Link is activated when App added to an Audiobus or IAA session while not playing in the background (if supported).

- Open LinkHut, press **Play**, and set Link to **Enabled**.
- Open App and set Link to **Enabled**.**&rArr;** The Link settings should indicate 1
  connected App.
- Make sure the app is not playing and switch to Audiobus
- Add the App as **Input** in Audiobus. Do this without tapping to wake it up. If the App
  is sleeping, switch back to it and then back to Audiobus and try again.
- Switch back to LinkHut **&rArr;** The Link settings should indicate 1 connected App.
- Switch back to Audiobus and eject the App from the Audiobus session
- Switch back to LinkHut **&rArr;** The Link settings should indicate 0 connected Apps.

**Note**: When an App in the background has deactivated Link, it must re-activate it if
it becomes part of an Audiobus or IAA session, even if does not come to the foreground.
Conversely, an App that is part of an Audiobus or IAA session session and is then
disconnected from the session while in the background and not playing should deactivate
Link.

## Promoting Link Integration

After investing the time and effort to add Link to your app, you will probably want to
tell the world about it. When you do so, please be sure to follow our [Ableton Link
promotion guidelines (PDF)](https://github.com/Ableton/LinkKit/blob/master/docs/Ableton_Link_Promotion.pdf).
The Link badge referred to in the guidelines can be found in the
[assets](https://github.com/Ableton/LinkKit/tree/master/assets) folder. You can also find
additional info and images in our [press kits](https://ableton.com/press) and use them as
you please.
