---
layout: link
active_page: link
title: Link Documentation
---

# Ableton Link

[Ableton Link](https://ableton.com/link) is a new technology that synchronizes musical beat, tempo, and phase across multiple applications running on one or more devices. Applications on devices connected to a local network discover each other automatically and form a musical session in which each participant can perform independently: anyone can start or stop while still staying in time. Anyone can change the tempo, the others will follow. Anyone can join or leave without disrupting the session.

Ableton provides two options for developers interested in integrating Link into their musical applications:

- [LinkKit](https://ableton.github.io/linkkit) SDK for iOS

- [Link](https://github.com/Ableton/link) cross-platform source code library

This page contains a discussion of fundamental Link concepts that apply to both of these offerings. For more detailed documentation and licensing information, please visit the appropriate project page linked above.

## Link Concepts

Link is different from other approaches to synchronizing electronic instruments that you may be familiar with. It is not designed to orchestrate multiple instruments so that they play together in lock-step along a shared timeline. In fact, Link-enabled apps each have their own independent timelines. The Link library maintains a temporal relationship between these independent timelines that provides the experience of playing in time without the timelines being identical.

Playing "in time" is an intentionally vague term that might have different meanings for different musical contexts and different applications. As a developer, you must decide the most natural way to map your application's musical concepts onto Link's synchronization model. For this reason, it's important to gain an intuitive understanding of how Link synchronizes **tempo**, **beat**, and **phase**.

### Tempo Synchronization

Tempo is a well understood parameter that represents the velocity of a beat timeline with respect to real time, giving it a unit of beats/time. Tempo synchronization is achieved when the beat timelines of all participants in a session are advancing at the same rate.

With Link, any participant can propose a change to the session tempo at any time. No single participant is responsible for maintaining the shared session tempo. Rather, each participant chooses to adopt the last tempo value that they've seen proposed on the network. This means that it is possible for participants' tempi to diverge during periods of tempo modification (especially during simultaneous modification by multiple participants), but this state is only temporary. The session will converge quickly to a common tempo after any modification. The Link approach to tempo relies on group adaptation to changes made by independent, autonomous actors - much like a group of traditional instrumentalists playing together.

### Beat Alignment

It's conceivable that for certain musical situations, participants would wish to only synchronize tempo and not other musical parameters. But for the vast majority of cases, playing with synchronized tempo in the absence of beat alignment would not be perceived as playing "in time." In this scenario, participants' beat timelines would advance at the same rate, but the relationship between values on those beat timelines would be undefined.

In most cases, we want to provide a stronger timing property for a session than just tempo synchronization - we also want beat alignment. When a session is in a state of beat alignment, an integral value on any participant's beat timeline corresponds to an integral value on all other participants' beat timelines. This property says nothing about the magnitude of beat values on each timeline, which can be different, just that any two timelines must only differ by an integral offset. For example, beat 1 on one participant's timeline might correspond to beat 3 or beat 4 on another's, but it cannot correspond to beat 3.5.

Note that in order for a session to maintain a state of beat alignment, it must have synchronized tempo.

### Phase Synchronization

Beat alignment is a necessary condition for playing "in time" in most circumstances, but it's often not enough. When working with bars or loops, a user may expect that the beat position within such a construct (the phase) be synchronized, resulting in alignment of bar or loop boundaries across participants.

In order to enable the desired bar and loop alignment, an application provides a quantum value to Link that specifies, in beats, the desired unit of phase synchronization. Link guarantees that session participants with the same quantum value will be phase aligned, meaning that if two participants have a 4 beat quantum, beat 3 on one participant's timeline could correspond to beat 11 on another's, but not beat 12. It also guarantees the expected relationship between sessions in which one participant has a multiple of another's quantum. So if one app has an 8-beat loop with a quantum of 8 and another has a 4-beat loop with a quantum of 4, then the beginning of an 8-beat loop will always correspond to the beginning of a 4-beat loop, whereas a 4-beat loop may align with the beginning or the middle of an 8-beat loop.

Specifying the quantum value and the handling of phase synchronization is the aspect of Link integration that leads to the greatest diversity of approaches among developers. There's no one-size-fits-all recommendation about how to do this, it is very application-specific. Some applicaitons have a constant quantum that never changes. Others allow it to change to match a changing value in their app, such as loop length or time signature. In Ableton Live, it is directly tied to the "Global Quantization" control, so it may be useful to explore how different values affect the behavior of Live in order to gain intuition about the quantum.

In order to maintain phase synchronization, the vast majority of Link-enabled applications (including Live) perform a quantized launch when the user starts transport. This means that the user sees some sort of count-in animation or flashing play button until starting at the next quantum boundary. This is a very satisfying interaction because it allows multiple users on different devices to start exactly together just by pressing play at roughly the same time. We strongly recommend that developers implement quantized launching in Link-enabled applications.
