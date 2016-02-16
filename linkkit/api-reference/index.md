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

Provides zero configuration peer discovery on a local wired or wifi network between multiple instances running on multiple devices. When peers are connected in a link session, they share a common tempo and quantized beat grid.

Each instance of the library has its own beat timeline that starts when the library is initialized and runs until the library instance is destroyed. Clients can reset the beat timeline in order to align it with an app's beat position when starting playback.

### Typedefs

[`ABLLinkRef`](#abllinkref)<br>
Reference to an instance of the library.

[`ABLLinkSessionTempoCallback`](#abllinksessiontempocallback)<br>
Called if Session Tempo changes.

[`ABLLinkIsEnabledCallback`](#abllinkisenabledcallback)<br>
Called if isEnabled state changes.

### Functions

[`ABLLinkNew`](#abllinknew)<br>
Initialize the library, providing an initial tempo and sync quantum.

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
Invoked on the main thread when the user changes the enabled state of the library via the Link settings view.

[`ABLLinkProposeTempo`](#abllinkproposetempo)<br>
Propose a new tempo to the link session.

[`ABLLinkGetSessionTempo`](#abllinkgetsessiontempo)<br>
Get the current tempo for the link session in Beats Per Minute.

[`ABLLinkBeatTimeAtHostTime`](#abllinkbeattimeathosttime)<br>
Conversion function to determine which value on the beat timeline should be hitting the device's output at the given host time.

[`ABLLinkHostTimeAtBeatTime`](#abllinkhosttimeatbeattime)<br>
Conversion function to determine which host time at the device's output represents the given beat time value.

[`ABLLinkResetBeatTime`](#abllinkresetbeattime)<br>
Reset the beat timeline with a desire to map the given beat time to the given host time, returning the actual beat time value that maps to the given host time.

[`ABLLinkSetQuantum`](#abllinksetquantum)<br>
Set the value used for quantization to the shared beat grid.

[`ABLLinkGetQuantum`](#abllinkgetquantum)<br>
Get the value currently being used by the system for quantization to the shared beat grid.

[`ABLLinkPhase`](#abllinkphase)<br>
Get the phase for a given beat time value on the shared beat grid with respect to the given quantum.

#### `ABLLinkRef`

Reference to an instance of the library.

<code class="is-block"><span>typedef struct ABLLink*</span> ABLLinkRef;</code>

#### `ABLLinkSessionTempoCallback`

Called if Session Tempo changes.

<code class="is-block"><span>typedef void</span> ( <span>*</span>ABLLinkSessionTempoCallback)(
    <span>double</span> sessionTempo,
    <span>void *</span>context);
</code>

**Parameters:**

- `sessionTempo`: User-visible representation of the session tempo as described in [`ABLLinkGetSessionTempo`](#abllinkgetsessiontempo)

#### `ABLLinkIsEnabledCallback`

Called if isEnabled state changes.

<code class="is-block"><span>typedef void</span> ( *ABLLinkIsEnabledCallback)(
    <span>bool</span> isEnabled,
    <span>void</span> *context);
</code>

**Parameters:**

- `isEnabled`: Whether Link is currently enabled

#### `ABLLinkNew`

Initialize the library, providing an initial tempo and sync quantum.

<code class="is-block"><span>ABLLinkRef</span> ABLLinkNew(
    <span>double</span> initialBpm,
    <span>double</span> syncQuantum);
</code>

The sync quantum is a value in beats that represents the granularity of synchronizaton with the shared quantization grid. A reasonable default value would be 1, which would guarantee that beat onsets would be synchronized with the session. Higher values would provide phase synchronization across multiple beats. For example, a value of 4 would cause this instance to be aligned to a 4/4 bar with any other instances in the session that have a quantum of 4 (or a multiple of 4).

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

When Link is active, it advertises itself on the local network and initiates connections with other peers. It is active by default after init.

#### `ABLLinkIsEnabled`

Is Link currently enabled by the user?

<code class="is-block"><span>bool</span> ABLLinkIsEnabled(
    ABLLinkRef);
</code>

The enabled status is only controllable by the user via the Link settings dialog and is not controllable programmatically.

#### `ABLLinkIsConnected`

Is Link currently connected to other peers?

<code class="is-block"><span>bool</span> ABLLinkIsConnected(
    ABLLinkRef);
</code>

#### `ABLLinkSetSessionTempoCallback`

Invoked on the main thread when the tempo of the Link session changes.

<code class="is-block"><span>void</span> ABLLinkSetSessionTempoCallback(
    ABLLinkRef,
    <span>ABLLinkSessionTempoCallback</span> callback,
    <span>void</span> *context);
</code>

#### `ABLLinkSetIsEnabledCallback`

Invoked on the main thread when the user changes the enabled state of the library via the Link settings view.

<code class="is-block"><span>void</span> ABLLinkSetIsEnabledCallback(
    ABLLinkRef,
    <span>ABLLinkIsEnabledCallback</span> callback,
    <span>void</span> *context);
</code>

#### `ABLLinkProposeTempo`

Propose a new tempo to the link session.

<code class="is-block"><span>void</span> ABLLinkProposeTempo(
    ABLLinkRef,
    <span>double</span> bpm,
    <span>uint64_t</span> hostTimeAtOutput);
</code>

**Parameters:**

- `bpm`: The new tempo to be used by the session.
- `hostTimeAtOutput`: The host time at which the change should occur. If the host time is too far in the past or future, the proposal may be rejected.

#### `ABLLinkGetSessionTempo`

Get the current tempo for the link session in Beats Per Minute.

<code class="is-block"><span>double</span> ABLLinkGetSessionTempo(
    ABLLinkRef);
</code>

This is a stable value that is appropriate for display to the user (unlike the value derived for a given audio buffer, which will vary due to clock drift, latency compensation, etc.)

#### `ABLLinkBeatTimeAtHostTime`

Conversion function to determine which value on the beat timeline should be hitting the device's output at the given host time.

<code class="is-block"><span>double</span> ABLLinkBeatTimeAtHostTime(
    ABLLinkRef,
    <span>uint64_t</span> hostTimeAtOutput);
</code>

In order to determine the host time at the device output, the AVAudioSession outputLatency property must be taken into consideration along with any additional buffering latency introduced by the software. This function guarantees a proportional relationship between hostTimeAtOutput and the resulting beat time: hostTime_2 > hostTime_1 => beatTime_2 > beatTime_1 when called twice from the same thread.

#### `ABLLinkHostTimeAtBeatTime`

Conversion function to determine which host time at the device's output represents the given beat time value.

<code class="is-block"><span>uint64_t</span> ABLLinkHostTimeAtBeatTime(
    ABLLinkRef,
    <span>double</span> beatTime);
</code>

This function does not guarantee a backwards conversion of the value returned by ABLLinkBeatTimeAtHostTime.

#### `ABLLinkResetBeatTime`

Reset the beat timeline with a desire to map the given beat time to the given host time, returning the actual beat time value that maps to the given host time.

<code class="is-block"><span>double</span> ABLLinkResetBeatTime(
    ABLLinkRef,
    <span>double</span> beatTime,
    <span>uint64_t</span> hostTimeAtOutput);
</code>

The returned value will differ from the requested beat time by up to a quantum due to quantization, but will always be less than or equal to the given beat time.

#### `ABLLinkSetQuantum`

Set the value used for quantization to the shared beat grid.

<code class="is-block"><span>void</span> ABLLinkSetQuantum(
    ABLLinkRef,
    <span>double</span> quantum);
</code>

**Parameters:**

- `quantum` in beats

The quantum value set here will be used when joining a session and whenresetting the beat timeline with ABLLinkResetBeatTime. It doesn't affect the results of the beat time / host time conversion functions and therefore will not cause a beat time jump if invoked while playing.

#### `ABLLinkGetQuantum`

Get the value currently being used by the system for quantization to the shared beat grid.

<code class="is-block"><span>double</span> ABLLinkGetQuantum(
    ABLLinkRef);
</code>

#### `ABLLinkPhase`

Get the phase for a given beat time value on the shared beat grid with respect to the given quantum.

<code class="is-block"><span>double</span> ABLLinkPhase(
    ABLLinkRef,
    <span>double</span> beatTime,
    <span>double</span> quantum);
</code>

The beat timeline exposed by the ABLLink functions are aligned to the shared beat grid according to the quantum value that was set at initialization or at the last call to ABLLinkResetBeatTime. This function allows access to the phase of beat time values with respect to other quanta. The returned value will be in the range [0, quantum).

## ABLLinkSettingsViewController.h
Copyright 2016, Ableton AG, Berlin. All rights reserved.

#### `ABLLinkSettingsViewController : UIViewController`

Settings view controller that provides users with the ability to view Link status and modify Link-related settings. Clients can integrate this view controller into their GUI as they see fit, but it is recommended that it be presented as a popover.

#### `+instance:`

Class method that provides an instance of the view controller given an ABLLink instance.

<code class="is-block">+ (<span>id</span>)instance:(<span>ABLLinkRef</span>)ablLink;
</code>

Clients must ensure that the ABLLink instance is not destroyed before the view controller.

<hr>

<span class="text-grey">Last Updated: Tuesday, January 26, 2016</span>
