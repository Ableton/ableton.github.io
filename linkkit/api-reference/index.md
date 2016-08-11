---
layout: linkkit
active_page: api-reference
title: LinkKit API Reference
---

# API Reference

## ABLLink.h
Copyright 2016, Ableton AG, Berlin. All rights reserved.

### Introduction

Cross-device shared tempo and quantized beat grid API for iOS.

Provides zero configuration peer discovery on a local wired or wifi
network between multiple instances running on multiple devices. When
peers are connected in a link session, they share a common tempo and
quantized beat grid.

Each instance of the library has its own beat timeline that starts
when the library is initialized and runs until the library instance is
destroyed. Clients can reset the beat timeline in order to align it
with an app's beat position when starting playback.

The library provides one timeline capture/commit function pair for use
in the audio thread and one for the main application thread. In
general, modifying the Link timeline should be done in the audio
thread for the most accurate timing results. The ability to modify the
Link timeline from application threads should only be used in cases
where an application's audio thread is not actively running or if it
doesn't generate audio at all. Modifying the Link timeline from both
the audio thread and an application thread concurrently is not advised
and will potentially lead to unexpected behavior.

### Typedefs

[`ABLLinkRef`](#abllinkref)<br>
Reference to an instance of the library.

[`ABLLinkTimelineRef`](#abllinktimelineref)<br>
A reference to a representation of a mapping between time and beats
for varying quanta.

[`ABLLinkSessionTempoCallback`](#abllinksessiontempocallback)<br>
Called if Session Tempo changes.

[`ABLLinkIsEnabledCallback`](#abllinkisenabledcallback)<br>
Called if isEnabled state changes.

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

[`ABLLinkSetSessionTempoCallback`](#abllinksetsessiontempocallback)<br>
Invoked on the main thread when the tempo of the Link session changes.

[`ABLLinkSetIsEnabledCallback`](#abllinksetisenabledcallback)<br>
Invoked on the main thread when the user changes the enabled state of
the library via the Link settings view.

[`ABLLinkSetIsConnectedCallback`](#abllinksetisconnectedcallback)<br>
Invoked on the main thread when the isConnected state of the library
changes.

[`ABLLinkCaptureAudioTimeline`](#abllinkcaptureaudiotimeline)<br>
Capture the current Link timeline from the audio thread.

[`ABLLinkCommitAudioTimeline`](#abllinkcommitaudiotimeline)<br>
Commit the given timeline to the Link session from the audio thread.

[`ABLLinkCaptureAppTimeline`](#abllinkcaptureapptimeline)<br>
Capture the current Link timeline from the main application thread.

[`ABLLinkCommitAppTimeline`](#abllinkcommitapptimeline)<br>
Commit the given timeline to the Link session from the main
application thread.

[`ABLLinkGetTempo`](#abllinkgettempo)<br>
The tempo of the given timeline, in Beats Per Minute.

[`ABLLinkSetTempo`](#abllinksettempo)<br>
Set the timeline tempo to the given bpm value, taking effect at the
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

#### `ABLLinkRef`

Reference to an instance of the library.

<code class="is-block"><span>typedef struct ABLLink*</span> ABLLinkRef;</code>

#### `ABLLinkTimelineRef`

A reference to a representation of a mapping between time and beats
for varying quanta.

<code class="is-block"><span>typedef struct ABLLinkTimeline*</span>
ABLLinkTimelineRef;</code>

#### `ABLLinkSessionTempoCallback`

Called if Session Tempo changes.

<code class="is-block"><span>typedef void</span> (
<span>*</span>ABLLinkSessionTempoCallback)( <span>double</span> sessionTempo, <span>void
*</span>context); </code>

**Parameters:**

- `sessionTempo`: User-visible representation of the session tempo as described in
  [`ABLLinkGetSessionTempo`](#abllinkgetsessiontempo)

#### `ABLLinkIsEnabledCallback`

Called if isEnabled state changes.

<code class="is-block"><span>typedef void</span> ( *ABLLinkIsEnabledCallback)(
    <span>bool</span> isEnabled,
    <span>void</span> *context);
</code>

**Parameters:**

- `isEnabled`: Whether Link is currently enabled

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

#### `ABLLinkSetSessionTempoCallback`

Invoked on the main thread when the tempo of the Link session changes.

<code class="is-block"><span>void</span> ABLLinkSetSessionTempoCallback(
    ABLLinkRef,
    ABLLinkSessionTempoCallback callback,
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

#### `ABLLinkSetIsConnectedCallback`

Invoked on the main thread when isConnected state of the library
changes.

<code class="is-block"><span>void</span> ABLLinkSetIsConnectedCallback(
    ABLLinkRef,
    ABLLinkIsConnectedCallback callback,
    <span>void</span> *context);
</code>

#### `ABLLinkCaptureAudioTimeline`

Capture the current Link timeline from the audio thread.

<code class="is-block">ABLLinkTimelineRef ABLLinkCaptureAudioTimeline(
    ABLLinkRef);
</code>

This function is lockfree and should ONLY be called in the audio
thread. It must not be accessed from any other threads. The returned
reference refers to a snapshot of the current Link state, so it should
be captured and used in a local scope. Storing the Timeline for later
use in a different context is not advised because it will provide an
outdated view on the Link state.

#### `ABLLinkCommitAudioTimeline`

Commit the given timeline to the Link session from the audio thread.

<code class="is-block"><span>void</span> ABLLinkCommitAudioTimeline(
    ABLLinkRef,
    ABLLinkTimelineRef);
</code>

This function is lockfree and should ONLY be called in the audio
thread. The given timeline will replace the current Link
timeline. Modifications to the session based on the new timeline will
be communicated to other peers in the session.

#### `ABLLinkCaptureAppTimeline`

Capture the current Link timeline from the main application thread.

<code class="is-block">ABLLinkTimelineRef ABLLinkCaptureAppTimeline(
    ABLLinkRef);
</code>

This function provides the ability to query the Link timeline from the
main application thread and should only be used from that thread. The
returned Timeline stores a snapshot of the current Link state, so it
should be captured and used in a local scope. Storing the Timeline for
later use in a different context is not advised because it will
provide an outdated view on the Link state.

#### `ABLLinkCommitAppTimeline`

Commit the timeline to the Link session from the main application thread.

<code class="is-block"><span>void</span> ABLLinkCommitAppTimeline(
    ABLLinkRef,
    ABLLinkTimelineRef);
</code>

This function should ONLY be called in the main thread. The given
timeline will replace the current Link timeline. Modifications to the
session based on the new timeline will be communicated to other peers
in the session.

#### `ABLLinkGetTempo`

The tempo of the given timeline, in Beats Per Minute.

<code class="is-block"><span>double</span> ABLLinkGetTempo(
    ABLLinkTimelineRef);
</code>

This is a stable value that is appropriate for display to the
user. Beat time progress will not necessarily match this tempo exactly
because of clock drift compensation.

#### `ABLLinkSetTempo`

Set the timeline tempo to the given bpm value, taking effect at the
given host time.

<code class="is-block"><span>void</span> ABLLinkSetTempo(
    ABLLinkTimelineRef,
    <span>double</span> bpm,
    <span>uint64_t</span> hostTimeAtOutput);
</code>

#### `ABLLinkBeatAtTime`

Get the beat value corresponding to the given host time for the given
quantum.

<code class="is-block"><span>double</span> ABLLinkBeatAtTime(
    ABLLinkTimelineRef,
    <span>uint64_t</span> hostTimeAtOutput,
    <span>double</span> quantum);
</code>

#### `ABLLinkTimeAtBeat`

Get the host time at which the sound corresponding to the given beat
time and quantum reaches the device's audio output.

<code class="is-block"><span>uint64_t</span> ABLLinkTimeAtBeat(
    ABLLinkTimelineRef,
    <span>double</span> beatTime,
    <span>double</span> quantum);
</code>

The inverse of ABLLinkBeatAtTime, assuming a constant tempo.<br>
`ABLLinkBeatAtTime(tl, ABLLinkTimeAtBeat(tl, b, q), q) == b`.

#### `ABLLinkPhaseAtTime`

Get the phase for a given beat time value on the shared beat grid with
respect to the given quantum.

<code class="is-block"><span>double</span> ABLLinkPhaseAtTime(
    ABLLinkTimelineRef,
    <span>uint64_t</span> hostTimeAtOutput,
    <span>double</span> quantum);
</code>

This function allows access to the phase of a host time as described above with respect
to a quantum. The returned value will be in the range `[0, quantum]`.

#### `ABLLinkRequestBeatAtTime`

Attempt to map the given beat time to the given host time in the
context of the given quantum.

<code class="is-block"><span>void</span> ABLLinkRequestBeatAtTime(
    ABLLinkTimelineRef,
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
    ABLLinkTimelineRef,
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
