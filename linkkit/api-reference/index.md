---
layout: linkkit
active_page: api-reference
title: LinkKit API Reference
---

# API Reference

## ABLLink.h
Copyright 2018, Ableton AG, Berlin. All rights reserved.

### Introduction

Cross-device shared tempo and quantized beat grid API for iOS.

Provides zero configuration peer discovery on a local wired or wifi
network between multiple instances running on multiple devices. When
peers are connected in a Link session, they share a common tempo and
quantized beat grid.
Optionally, start/stop commands can be shared among a subgroup of peers.

Each instance of the library has its own session state describing a beat timeline
and a start/stop state. The timeline starts when the library is initialized and
runs until the library instance is destroyed. Clients can reset the beat timeline in
order to align it with an app's beat position when starting playback.
The start/stop state does not stop the timeline from progressing but rather describes
the users intent to be playing.

The library provides one session state capture/commit function pair for use
in the audio thread and one for the main application thread. In
general, modifying the Link session state should be done in the audio
thread for the most accurate timing results. The ability to modify the
Link session state from application threads should only be used in cases
where an application's audio thread is not actively running or if it
doesn't generate audio at all. Modifying the Link session state from both
the audio thread and an application thread concurrently is not advised
and will potentially lead to unexpected behavior.

### Typedefs

[`ABLLinkRef`](#abllinkref)<br>
Reference to an instance of the library.

[`ABLLinkSessionStateRef`](#abllinksessionstateref)<br>
A reference to a representation of the current Session State.

[`ABLLinkSessionTempoCallback`](#abllinksessiontempocallback)<br>
Called if Session Tempo changes.

[`ABLLinkIsEnabledCallback`](#abllinkisenabledcallback)<br>
Called if isEnabled state changes.

[`ABLLinkIsStartStopSyncEnabledCallback`](#abllinkisstartstopsyncenabledcallback)<br>
Called if the user enables or disables Start Stop Sync.

[`ABLLinkIsConnectedCallback`](#abllinkisconnectedcallback)<br>
Called if isConnected state changes.

### Functions

[`ABLLinkNew`](#abllinknew)<br>
Initialize the library, providing an initial tempo.

[`ABLLinkDelete`](#abllinkdelete)<br>
Destroy the library instance and cleanup its associated resources.

[`ABLLinkSetActive`](#abllinksetactive)<br>
Set whether Link should be active or not.

[`ABLLinkIsEnabled`](#abllinkisenabled)<br>
Is Link currently enabled by the user?

[`ABLLinkIsConnected`](#abllinkisconnected)<br>
Is Link currently connected to other peers?

[`ABLLinkIsStartStopSyncEnabled`](#abllinkisstartstopsyncenabled)<br>
Is Start Stop Sync enabled by the user?
To allow the user to enable Start Stop Sync a Boolean entry YES under the key
ABLLinkStartStopSyncSupported must be added to Info.plist.

[`ABLLinkSetSessionTempoCallback`](#abllinksetsessiontempocallback)<br>
Invoked on the main thread when the tempo of the Link session changes.

[`ABLLinkSetStartStopCallback`](#abllinksetstartstopcallback)<br>
Invoked on the main thread when the start/stop state changes.

[`ABLLinkSetIsEnabledCallback`](#abllinksetisenabledcallback)<br>
Invoked on the main thread when the user changes the enabled state of
the library via the Link settings view.

[`ABLLinkSetIsStartStopSyncEnabledCallback`](@abllinksetisstartstopsyncenabledcallback)<br>
Invoked on the main thread when the user changes the Start Stop Sync enabled state of the
library via the Link settings view.
To allow the user to enable Start Stop Sync a Boolean entry YES under the key
ABLLinkStartStopSyncSupported must be added to Info.plist.

[`ABLLinkSetIsConnectedCallback`](#abllinksetisconnectedcallback)<br>
Invoked on the main thread when the isConnected state of the library
changes.

[`ABLLinkCaptureAudioSessionState`](#abllinkcaptureaudiosessionstate)<br>
Capture the current Link session state from the audio thread.

[`ABLLinkCommitAudioSessionState`](#abllinkcommitaudiosessionstate)<br>
Commit the given session state to the Link session from the audio thread.

[`ABLLinkCaptureAppSessionState`](#abllinkcaptureappsessionstate)<br>
Capture the current Link session state from the main application thread.

[`ABLLinkCommitAppSessionState`](#abllinkcommitappsessionstate)<br>
Commit the given session state to the Link session from the main
application thread.

[`ABLLinkGetTempo`](#abllinkgettempo)<br>
The tempo of the given session state, in Beats Per Minute.

[`ABLLinkSetTempo`](#abllinksettempo)<br>
Set the session state tempo to the given bpm value, taking effect at the
given host time.

[`ABLLinkBeatAtTime`](#abllinkbeatattime)<br>
Get the beat value corresponding to the given host time for the given
quantum.

[`ABLLinkTimeAtBeat`](#abllinktimeatbeat)<br>
Get the host time at which the sound corresponding to the given beat
time and quantum reaches the device's audio output.

[`ABLLinkPhaseAtTime`](#abllinkphaseattime)<br>
Get the phase for a given beat time value on the shared beat grid with
respect to the given quantum.

[`ABLLinkRequestBeatAtTime`](#abllinkrequestbeatattime)<br>
Attempt to map the given beat time to the given host time in the
context of the given quantum.

[`ABLLinkForceBeatAtTime`](#abllinkforcebeatattime)<br>
Rudely re-map the beat/time relationship for all peers in a session.

[`ABLLinkSetIsPlaying`](#abllinksetisplaying)<br>
Set if transport should be playing or stopped.

[`ABLLinkIsPlaying`](#abllinkisplaying)<br>
Is transport playing?

[`ABLLinkRequestBeatAtStartPlayingTime`](#abllinkrequestbeatatsartplayingtime)<br>
Attempt to map the given beat time to the time transport was started.

[`ABLLinkSetIsPlayingAndRequestBeatAtTime`](#abllinksetisplayingandrequestbeatattime)<br>
Attempt to start playing at the given beat time and host time.

#### `ABLLinkRef`

Reference to an instance of the library.

<code class="is-block"><span>typedef struct ABLLink*</span> ABLLinkRef;</code>

#### `ABLLinkSessionStateRef`

A session state represents a timeline and a start/stop state. The timeline is a representation of a mapping between time and beats for varying quanta. The start/stop state represents the user intention to start or stop transport at a specific time. Start stop synchronization is an optional feature that allows to share the user request to start or stop transport between a subgroup of peers in a Link session. When observing a change of start/stop state, audio playback of a peer should be started or stopped the same way it would have happened if the user had requested that change at the according time locally. The start/stop state can only be changed by the user. This means that the current local start/stop state persists when joining or leaving a Link session. After joining a Link session start/stop change requests will be communicated to all connected peers.

<code class="is-block"><span>typedef struct ABLLinkSessionState*</span>
ABLLinkSessionStateRef;</code>

#### `ABLLinkSessionTempoCallback`

Called if Session Tempo changes.

<code class="is-block"><span>typedef void</span> (
<span>*</span>ABLLinkSessionTempoCallback)( <span>double</span> sessionTempo, <span>void
*</span>context); </code>

**Parameters:**

- `sessionTempo`: User-visible representation of the session tempo as described in
  [`ABLLinkGetSessionTempo`](#abllinkgetsessiontempo)

#### `ABLLinkStartStopCallback`

Called if Session transport start/stop state changes.

<code class="is-block"><span>typedef void</span> (
<span>*</span>ABLLinkStartStopCallback)(<span>bool</span> isPlaying, <span>void</span> *context); </code>

**Parameters**

- `isPlaying` Whether Link is currently playing.

#### `ABLLinkIsEnabledCallback`

Called if isEnabled state changes.

<code class="is-block"><span>typedef void</span> ( *ABLLinkIsEnabledCallback)(
    <span>bool</span> isEnabled,
    <span>void</span> *context);
</code>

**Parameters:**

- `isEnabled`: Whether Link is currently enabled

#### `ABLLinkIsStartStopSyncEnabledCallback`

Called if IsStartStopSyncEnabled state changes.

<code class="is-block"><span>typedef void</span>(*ABLLinkIsStartStopSyncEnabledCallback)(
  <span>bool</span> isEnabled,
  <span>void</span> *context);
</code>

**Parameters:**

- `isEnabled`: Whether Start Stop Sync is currently enabled

#### `ABLLinkIsConnectedCallback`

Called if isConnected state changes.

<code class="is-block"><span>typedef void</span> ( *ABLLinkIsConnectedCallback)(
    <span>bool</span> isConnected,
    <span>void</span> *context);
</code>

**Parameters:**

- `isConnected`: Whether Link is currently connected to other peers

#### `ABLLinkNew`

Initialize the library, providing an initial tempo.

<code class="is-block">ABLLinkRef ABLLinkNew(
    <span>double</span> initialBpm);
</code>

#### `ABLLinkDelete`

Destroy the library instance and cleanup its associated resources.

<code class="is-block"><span>void</span> ABLLinkDelete(
    ABLLinkRef);
</code>

#### `ABLLinkSetActive`

Set whether Link should be active or not.

<code class="is-block"><span>void</span> ABLLinkSetActive(
    ABLLinkRef,
    <span>bool</span> active);
</code>

When Link is active, it advertises itself on the local network and
initiates connections with other peers. It is active by default after
init.

#### `ABLLinkIsEnabled`

Is Link currently enabled by the user?

<code class="is-block"><span>bool</span> ABLLinkIsEnabled(
    ABLLinkRef);
</code>

The enabled status is only controllable by the user via the Link
settings dialog and is not controllable programmatically.

#### `ABLLinkIsConnected`

Is Link currently connected to other peers?

<code class="is-block"><span>bool</span> ABLLinkIsConnected(
    ABLLinkRef);
</code>

#### `ABLLinkIsStartStopSyncEnabled`

Is Start Stop Sync currently enabled by the user?

<code class="is-block"><span>bool</span> ABLLinkIsStartStopSyncEnabled(ABLLinkRef);</code>

#### `ABLLinkSetSessionTempoCallback`

Invoked on the main thread when the tempo of the Link session changes.

<code class="is-block"><span>void</span> ABLLinkSetSessionTempoCallback(
    ABLLinkRef,
    ABLLinkSessionTempoCallback callback,
    <span>void</span> *context);
</code>

#### `ABLLinkSetStartStopCallback`

Invoked on the main thread when the start/stop state of the Link session changes.

<code class="is-block"><span>void</span> ABLLinkSetStartStopCallback(
    ABLLinkRef,
    ABLLinkStartStopCallback callback,
    <span>void</span> *context);
</code>

#### `ABLLinkSetIsEnabledCallback`

Invoked on the main thread when the user changes the enabled state of
the library via the Link settings view.

<code class="is-block"><span>void</span> ABLLinkSetIsEnabledCallback(
    ABLLinkRef,
    ABLLinkIsEnabledCallback callback,
    <span>void</span> *context);
</code>

#### `ABLLinkSetIsStartStopSyncEnabledCallback`

Invoked on the main thread when the user changes the Start Stop Sync enabled
state via the Link settings view.

<code class="is-block"><span>void</span>ABLLinkSetIsStartStopSyncEnabledCallback(
    ABLLinkRef,
    ABLLinkIsStartStopSyncEnabledCallback callback,
    <span>void</span> *context);
</code>

#### `ABLLinkSetIsConnectedCallback`

Invoked on the main thread when isConnected state of the library
changes.

<code class="is-block"><span>void</span> ABLLinkSetIsConnectedCallback(
    ABLLinkRef,
    ABLLinkIsConnectedCallback callback,
    <span>void</span> *context);
</code>

#### `ABLLinkCaptureAudioSessionState`

Capture the current Link session state from the audio thread.

<code class="is-block">ABLLinkSessionStateRef ABLLinkCaptureAudioSessionState(
    ABLLinkRef);
</code>

This function is lockfree and should ONLY be called in the audio thread. It must
not be accessed from any other threads. The returned reference refers to a
snapshot of the current session state, so it should be captured and used in a
local scope. Storing the session state for later use in a different context is
not advised because it will provide an outdated view on the Link state.

#### `ABLLinkCommitAudioSessionState`

Commit the given session state to the Link session from the audio thread.

<code class="is-block"><span>void</span> ABLLinkCommitAudioSessionState(
    ABLLinkRef,
    ABLLinkSessionStateRef);
</code>

This function is lockfree and should ONLY be called in the audio thread. The
given session state will replace the current Link session state. Modifications
to the session based on the new session state will be communicated to other
peers in the session.

#### `ABLLinkCaptureAppSessionState`

Capture the current Link session state from the main application thread.

<code class="is-block">ABLLinkSessionStateRef ABLLinkCaptureAppSessionState(
    ABLLinkRef);
</code>

This function provides the ability to query the Link session state from the main
application thread and should only be used from that thread. The returned
session state stores a snapshot of the current Link state, so it should be
captured and used in a local scope. Storing the session state for later use in a
different context is not advised because it will provide an outdated view on the
Link state.

#### `ABLLinkCommitAppSessionState`

Commit the session state to the Link session from the main application thread.

<code class="is-block"><span>void</span> ABLLinkCommitAppSessionState(
    ABLLinkRef,
    ABLLinkSessionStateRef);
</code>

This function should ONLY be called in the main thread. The given session state
will replace the current Link session state. Modifications to the session based
on the new session state will be communicated to other peers in the session.

#### `ABLLinkGetTempo`

The tempo of the given session state, in Beats Per Minute.

<code class="is-block"><span>double</span> ABLLinkGetTempo(
    ABLLinkSessionStateRef);
</code>

This is a stable value that is appropriate for display to the
user. Beat time progress will not necessarily match this tempo exactly
because of clock drift compensation.

#### `ABLLinkSetTempo`

Set the tempo to the given bpm value at the given time. The change is applied 
immediately and sent to the network after committing the session state.

<code class="is-block"><span>void</span> ABLLinkSetTempo(
    ABLLinkSessionStateRef,
    <span>double</span> bpm,
    <span>uint64_t</span> hostTimeAtOutput);
</code>

#### `ABLLinkBeatAtTime`

Get the beat value corresponding to the given host time for the given
quantum.

<code class="is-block"><span>double</span> ABLLinkBeatAtTime(
    ABLLinkSessionStateRef,
    <span>uint64_t</span> hostTimeAtOutput,
    <span>double</span> quantum);
</code>

#### `ABLLinkTimeAtBeat`

Get the host time at which the sound corresponding to the given beat
time and quantum reaches the device's audio output.

<code class="is-block"><span>uint64_t</span> ABLLinkTimeAtBeat(
    ABLLinkSessionStateRef,
    <span>double</span> beatTime,
    <span>double</span> quantum);
</code>

The inverse of ABLLinkBeatAtTime, assuming a constant tempo.<br>
`ABLLinkBeatAtTime(tl, ABLLinkTimeAtBeat(tl, b, q), q) == b`.

#### `ABLLinkPhaseAtTime`

Get the phase for a given beat time value on the shared beat grid with
respect to the given quantum.

<code class="is-block"><span>double</span> ABLLinkPhaseAtTime(
    ABLLinkSessionStateRef,
    <span>uint64_t</span> hostTimeAtOutput,
    <span>double</span> quantum);
</code>

This function allows access to the phase of a host time as described above with respect
to a quantum. The returned value will be in the range `[0, quantum)`.

#### `ABLLinkRequestBeatAtTime`

Attempt to map the given beat time to the given host time in the
context of the given quantum.

<code class="is-block"><span>void</span> ABLLinkRequestBeatAtTime(
    ABLLinkSessionStateRef,
    <span>double</span> beatTime,
    <span>uint64_t</span> hostTimeAtOutput,
    <span>double</span> quantum);
</code>

This function behaves differently depending on the state of the
session. If no other peers are connected, then this instance is in a
session by itself and is free to re-map the beat/time relationship
whenever it pleases.

If there are other peers in the session, this instance should not
abruptly re-map the beat/time relationship in the session because that
would lead to beat discontinuities among the other peers. In this
case, the given beat will be mapped to the next time value greater
than the given time with the same phase as the given beat.

This function is specifically designed to enable the concept of
"quantized launch" in client applications. If there are no other peers
in the session, then an event (such as starting transport) happens
immediately when it is requested. If there are other peers, however,
we wait until the next time at which the session phase matches the
phase of the event, thereby executing the event in-phase with the
other peers in the session. The client only needs to invoke this
method to achieve this behavior and should not need to explicitly
check the number of peers.

#### `ABLLinkForceBeatAtTime`

Rudely re-map the beat/time relationship for all peers in a session.

<code class="is-block"><span>void</span> ABLLinkForceBeatAtTime(
    ABLLinkSessionStateRef,
    <span>double</span> beatTime,
    <span>uint64_t</span> hostTimeAtOutput,
    <span>double</span> quantum);
</code>

DANGER: This function should only be needed in certain special
circumstances. Most applications should not use it. It is very similar
to ABLLinkRequestBeatAtTime except that it does not fall back to the
quantizing behavior when it is in a session with other peers. Calling
this method will unconditionally map the given beat time to the given
host time and broadcast the result to the session. This is very
anti-social behavior and should be avoided.

One of the few legitimate uses of this method is to synchronize a Link
session with an external clock source. By periodically forcing the
beat/time mapping according to an external clock source, a peer can
effectively bridge that clock into a Link session. Much care must be
taken at the application layer when implementing such a feature so
that users do not accidentally disrupt Link sessions that they may
join.

#### `ABLLinkSetIsPlaying`

Set if transport should be playing or stopped at the given time.

<code class="is-block"><span>void</span> ABLLinkSetIsPlaying(
  ABLLinkSessionStateRef,
  <span>bool</span> isPlaying,
  <span>uint64_t</span> hostTimeAtOutput);
</code>

#### `ABLLinkIsPlaying`

Is transport playing?

<code class="is-block"><span>bool<span>ABLLinkIsPlaying(ABLLinkSessionStateRef);
</code>

#### `ABLLinkTimeForIsPlaying`

Get the time at which a transport start/stop occurs

<code class="is-block"><span>uint64_t</span> ABLLinkTimeForIsPlaying(ABLLinkSessionStateRef);
</code>

#### `ABLLinkRequestBeatAtStartPlayingTime`

Convenience function to attempt to map the given beat to the time when transport
is starting to play in context to the given quantum. This function evaluates to
a no-op if ABLLinkIsPlaying() equals false.

<code class="is-block"><span>void</span> ABLLinkRequestBeatAtStartPlayingTime(
  ABLLinkSessionStateRef,
  <span>double</span> beatTime,
  <span>double</span> quantum);

#### `ABLLinkSetIsPlayingAndRequestBeatAtTime`

Convenience function to start or stop transport at a given time and attempt to
map the given beat to this time in context of the given quantum.

<code class="is-block"><span>void</span> ABLLinkSetIsPlayingAndRequestBeatAtTime(
  ABLLinkSessionStateRef,
  <span>bool</span> isPlaying,
  <span>uint64_t</span> hostTimeAtOutput,
  <span>double</span> beatTime,
  <span>double</span> quantum);
</code>

## ABLLinkSettingsViewController.h
Copyright 2016, Ableton AG, Berlin. All rights reserved.

#### `ABLLinkSettingsViewController : UIViewController`

Settings view controller that provides users with the ability to view Link status and
modify Link-related settings. Clients can integrate this view controller into their GUI
as they see fit, but it is recommended that it be presented as a popover.

#### `+instance:`

Class method that provides an instance of the view controller given an ABLLink instance.

<code class="is-block">+ (<span>id</span>)instance:(<span>ABLLinkRef</span>)ablLink;
</code>

Clients must ensure that the ABLLink instance is not destroyed before the view controller.

<hr>

<span class="text-grey">Last Updated: Tuesday, August 3, 2016</span>
