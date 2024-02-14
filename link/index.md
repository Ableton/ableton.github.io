---
layout: link
active_page: link
title: Link Documentation
---

# Ableton Link

[Ableton Link](https://ableton.com/link) is a technology that synchronizes musical beat,
tempo, phase, and start/stop commands across multiple applications running on one or more
devices. Applications on devices connected to a local network discover each other
automatically and form a musical session in which each participant can perform
independently: anyone can start or stop while still staying in time. Anyone can change
the tempo, the others will follow. Anyone can join or leave without disrupting the session.

Ableton provides two options for developers interested in integrating Link into their
musical applications:

- [LinkKit](/linkkit) SDK for iOS

- [Link](https://github.com/Ableton/link) cross-platform source code library

This page contains a discussion of fundamental Link concepts that apply to both of these
projects. For more detailed documentation and licensing information, please visit the
appropriate project page linked above.

## Link Concepts

Link is different from other approaches to synchronizing electronic instruments that you
may be familiar with. It is not designed to orchestrate multiple instruments so that they
play together in lock-step along a shared timeline. In fact, Link-enabled apps each have
their own independent timelines. The Link library maintains a temporal relationship
between these independent timelines that provides the experience of playing in time
without the timelines being identical.

Playing "in time" is an intentionally vague term that might have different meanings for
different musical contexts and different applications. As a developer, you must decide
the most natural way to map your application's musical concepts onto Link's
synchronization model. For this reason, it's important to gain an intuitive understanding
of how Link synchronizes **tempo**, **beat**, and **phase**.
Version 3 of Link makes it possible to share **start** and **stop** commands.

### Tempo Synchronization

Tempo is a well understood parameter that represents the velocity of a beat timeline with
respect to real time, giving it a unit of beats/time. Tempo synchronization is achieved
when the beat timelines of all participants in a session are advancing at the same rate.

With Link, any participant can propose a change to the session tempo at any time. No
single participant is responsible for maintaining the shared session tempo. Rather, each
participant chooses to adopt the last tempo value that they've seen proposed on the
network. This means that it is possible for participants' tempi to diverge during periods
of tempo modification (especially during simultaneous modification by multiple
participants), but this state is only temporary. The session will converge quickly to a
common tempo after any modification. The Link approach to tempo relies on group
adaptation to changes made by independent, autonomous actors - much like a group of
traditional instrumentalists playing together.

### Beat Alignment

It's conceivable that for certain musical situations, participants would wish to only
synchronize tempo and not other musical parameters. But for the vast majority of cases,
playing with synchronized tempo in the absence of beat alignment would not be perceived
as playing "in time." In this scenario, participants' beat timelines would advance at the
same rate, but the relationship between values on those beat timelines would be undefined.

In most cases, we want to provide a stronger timing property for a session than just
tempo synchronization - we also want beat alignment. When a session is in a state of beat
alignment, an integral value on any participant's beat timeline corresponds to an
integral value on all other participants' beat timelines. This property says nothing
about the magnitude of beat values on each timeline, which can be different, just that
any two timelines must only differ by an integral offset. For example, beat 1 on one
participant's timeline might correspond to beat 3 or beat 4 on another's, but it cannot
correspond to beat 3.5.

Note that in order for a session to maintain a state of beat alignment, it must have
synchronized tempo.

### Phase Synchronization

Beat alignment is a necessary condition for playing "in time" in most circumstances, but
it's often not enough. When working with bars or loops, a user may expect that the beat
position within such a construct (the phase) be synchronized, resulting in alignment of
bar or loop boundaries across participants.

In order to enable the desired bar and loop alignment, an application provides a quantum
value to Link that specifies, in beats, the desired unit of phase synchronization. Link
guarantees that session participants with the same quantum value will be phase aligned,
meaning that if two participants have a 4 beat quantum, beat 3 on one participant's
timeline could correspond to beat 11 on another's, but not beat 12. It also guarantees
the expected relationship between sessions in which one participant has a multiple of
another's quantum. So if one app has an 8-beat loop with a quantum of 8 and another has a
4-beat loop with a quantum of 4, then the beginning of an 8-beat loop will always
correspond to the beginning of a 4-beat loop, whereas a 4-beat loop may align with the
beginning or the middle of an 8-beat loop.

Specifying the quantum value and the handling of phase synchronization is the aspect of
Link integration that leads to the greatest diversity of approaches among developers.
There's no one-size-fits-all recommendation about how to do this, it is very
application-specific. Some applications have a constant quantum that never changes.
Others allow it to change to match a changing value in their app, such as loop length or
time signature. In Ableton Live, it is directly tied to the "Global Quantization"
control, so it may be useful to explore how different values affect the behavior of Live
in order to gain intuition about the quantum.

In order to maintain phase synchronization, the vast majority of Link-enabled
applications (including Live) perform a quantized launch when the user starts transport.
This means that the user sees some sort of count-in animation or flashing play button
until starting at the next quantum boundary. This is a very satisfying interaction
because it allows multiple users on different devices to start exactly together just by
pressing play at roughly the same time. We strongly recommend that developers implement
quantized launching in Link-enabled applications.

### Start/Stop Synchronization

As of Version 3, Link allows peers to share information on the user's intent to start or
stop transport with other peers that have the feature enabled.
Start/stop state changes only follow user actions. This means applications will not adapt
to, or automatically change the start/stop state of a Link session when they are joining.
After a peer joins a session it exposes and listens to all upcoming start/stop state
changes. This is different to tempo, beat, and phase that are automatically aligned as
soon as an application joins a session.
As every application handles start and stop commands according to its capabilities and
quantization, it is not expected that applications start or stop at the same time. Rather
every application should start according to its quantum and phase.

## Link API

The [Link](https://github.com/Ableton/link) repo contains C++ source code implementing
the Link protocol and a C++ API for integrating applications. Developers making iOS apps
should instead use the [LinkKit](https://ableton.github.io/linkkit) SDK for iOS, which
provides a pre-built library and a C/Objective-C API. The API provided by LinkKit is an
almost direct mapping of Link's C++ API into C. So while the APIs are distinct, they
share common concepts that we will explore in this section.

### Session State

In Link, a session state contains timeline information and a start/stop state.

A timeline is represented as a triple of `(beat, time, tempo)`, which defines a bijection
between the sets of all beat and time values. Converting between beats and time is the
most basic service that Link provides to integrating applications - an application will
generally want to know what beat value corresponds to a given moment in time. The timeline
implements this and all other timing-related queries and modifications available
to Link clients.

A start/stop state represents the user's intent to start or stop transport at a
given time.

Timeline and start/stop state may change over time. A session state value only
represents a snapshot of the state of the system at a particular moment. Link provides
clients the ability to 'capture' such a snapshot. This is the only mechanism for
obtaining a session state value.

Once a session state value is captured, clients may query its properties or modify it by
changing its tempo, its beat/time mapping or its start/stop state. Modifications to the
captured session state are *not* propagated to the Link session automatically - clients
must 'commit' the modified session state back to Link for it to take effect.

A major benefit of the capture-commit model for session states is that a captured
session state can be known to have a consistent value for the duration of a computation
in which it is used. If clients queried timing information from the Link object directly
without capturing a session state, the results could be inconsistent during the course
of a computation because of asynchronous changes coming from other participants in
the Link session.

### Session States and Threads

The audio thread, the thread that computes and fills audio buffers,
has special timing constraints that must be managed carefully. It's usually necessary to
query Link state data while computing audio, so the Link API provides a realtime-safe
session state capture/commit function pair. This allows clients to query and modify the
Link session state directly from the audio callback. It's important that this audio-thread
specific interface *only* be used from the audio thread.

It is also often convenient to be able to access the current Link session state from the
main thread or other application threads, for example when rendering the current beat time
in a GUI. For this reason, Link also provides a session state capture/commit function pair
to be used on application threads. These versions must not be used from the audio thread
as they may block.

While it is possible to commit session state modifications from an application thread,
this should generally be avoided by clients that implement a custom audio callback and use
Link from the audio thread. Because of the real-time constraints in the audio thread,
changes made to the Link session state from an application thread are processed
asynchronously and are not immediately visible to the audio thread. The same is true for
changes made from the audio thread - the new session state will eventually be visible to
the application thread, but clients cannot rely on exactly when. It's especially important
to take this into account when combining Link session state modifications with other
cross-thread communication mechanisms employed by the application. For example, if a
session state is committed from an application thread and in the next line an atomic flag
is set, the audio thread will almost certainly observe the flag being set before observing
the new session state value.

In order to avoid these complexities, it is recommended that applications that implement
a custom audio callback only modify the Link session state from the audio thread.
Application threads may query the session state but should not modify it. This approach
also leads to better timing accuracy because session state changes can be specified to
occur at buffer boundaries or even at specific samples, which is not possible from an
application thread.

## Resources

The following open source repositories integrate Link.
*Those are not Ableton repositories. In case of issues please contact the respective maintainer. 
If you are missing a repository in this list please open a pull request or mail to [link-devs@ableton.com](mailto:link-devs@ableton.com).*
- [aalink](https://github.com/artfwo/aalink)
- [abl_link](https://github.com/libpd/abl_link)
- [AndroidLinkAudio](https://github.com/jbloit/AndroidLinkAudio)
- [carabiner](https://github.com/brunchboy/carabiner)
- [Go bindings](https://github.com/antoi-ne/al)
- [JuceSampler](https://github.com/ianacaburian/AbletonLink_JuceSampler)
- [link-python](https://github.com/gonzaloflirt/link-python)
- [comoc/ofxAbletonLink](https://github.com/comoc/ofxAbletonLink)
- [UnityAbletonLink](https://github.com/comoc/UnityAbletonLink)
- [node-abletonlink](https://github.com/2bbb/node-abletonlink)
- [2bbb/ofxAbletonLink](https://github.com/2bbb/ofxAbletonLink)
- [lua-abletonlink](https://luarocks.org/modules/s-ol/abletonlink)
- [VL.IO.AbletonLink](https://github.com/SpaceMusicZH/VL.IO.AbletonLink)

The following links may provide useful information if you are trying to integrate Link
with your own applications:

- [Video presentation - "Ableton Link – A technology to synchronize music software"](https://media.ccc.de/v/lac2018-42-ableton_link_a_technology_to_synchronize_music_software)

- [Paper - "Ableton Link – A technology to synchronize music software"](https://depositonce.tu-berlin.de/bitstream/11303/7886/4/LAC2018_proceedings.pdf#page=51)

- [A list of Link enabled products](https://www.ableton.com/en/link/products/)

## Listing your product on ableton.com

If your product supports Link we will happily list it on [our website](https://ableton.com/link/products). To be added to the listing please send the following to [link-devs@ableton.com](mailto:link-devs@ableton.com)
- Icon in JPEG format: min. 400 x 400 pixels and 144 ppi
- Short product description: max. 200 characters including spaces
- Product website or app store link, pointing to your product


