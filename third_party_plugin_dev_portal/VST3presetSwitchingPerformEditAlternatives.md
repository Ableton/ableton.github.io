---
layout: link
active_page: VST3presetSwitchingPerformEditAlternatives.
title: Recommendations to VST3 plugin manufacturers regarding bulk calls to performEdit
---

# Recommendations to VST3 plugin manufacturers regarding bulk calls to performEdit
## Introduction

For some VST3 plugins, Ableton receives reports of user interface lag when users switch presets from the plugin editor. After investigation, it seems that some VST3 manufacturers implement preset changes from the VST3 editor by calling performEdit for all parameters in order to notify both the DAW and the VST3 controller counterpart of its new state.

Live needs time to react to performEdit for our parameter mapping system.
Unfortunately, we receive calls from performEdit synchronously, and cannot optimize for situations where many successive calls of performEdit will happen.

## Steinberg’s recommendations: editor uses private communication to send new values to processor

This solution is to use private communication (for example sending a custom IMessage) from editor to processor with all the new values to set before calling restartComponent and making sure that the controller will return the correct information before making a call to restartComponent(kParamValuesChanged).

See below some quotes from a Steinberg developer about it:

[https://forums.steinberg.net/t/how-to-use-restartcomponent-and-which-flags-are-the-right-one-when-changing-all-characteristics-parameters-except-size/202031/11](https://forums.steinberg.net/t/how-to-use-restartcomponent-and-which-flags-are-the-right-one-when-changing-all-characteristics-parameters-except-size/202031/11)

> Hi Erez, you don’t need to call for every parameter beginEdit, performEdit, endEdit. If you do this, then the parameter values are already known in the host and you don’t need to call restartComponent with kParamValuesChanged. If you do this only to tell your processor to use these new values, you can also send your processor a custom message with all the new values and set them yourself in the processor. You just have to make sure that before you make a call to restartComponent with `kParamTitlesChanged|kParamValuesChanged`, that your controller will return the correct information afterwards.

[https://forums.steinberg.net/t/how-to-use-restartcomponent-and-which-flags-are-the-right-one-when-changing-all-characteristics-parameters-except-size/202031/13](https://forums.steinberg.net/t/how-to-use-restartcomponent-and-which-flags-are-the-right-one-when-changing-all-characteristics-parameters-except-size/202031/13)

> If you now call restartComponent with `kParamTitlesChanged|kParamValuesChanged`, the host should have the new parameter names, flags and values. The host will not send the values to your processor in the process call this way. So if you need them there, but without going thru the beginEdit, performEdit, endEdit methods, you could allocate an IMessage with `IHostApplication::createInstance()` fill it with the necessary information and send it via `IConnectionPoint::notify` to your processor.

## Relevant VST3 documentation

### Optional separation of processor and edit controller

[https://steinbergmedia.github.io/vst3_dev_portal/pages/Technical+Documentation/API+Documentation/Index.html#basic-concept](https://steinbergmedia.github.io/vst3_dev_portal/pages/Technical+Documentation/API+Documentation/Index.html#basic-concept)

> The design of VST 3 suggests a complete separation of processor and edit controller by implementing two components. […] A plug-in that supports this separation has to set the Steinberg::Vst::kDistributable flag in the class info of the processor component (Steinberg::PClassInfo2::classFlags). Of course not every plug-in can support this, for example if it depends deeply on resources that cannot be moved easily to another computer. So when this flag is not set, the host must not try to separate the components in any way.

### private communication (custom IMessage)

[https://steinbergmedia.github.io/vst3_dev_portal/pages/Technical+Documentation/API+Documentation/Index.html#private-communication](https://steinbergmedia.github.io/vst3_dev_portal/pages/Technical+Documentation/API+Documentation/Index.html#private-communication)

> Data that is unknown to the host can be transmitted by means of messages. The communication interfaces are
> * Steinberg::Vst::IConnectionPoint: The host establishes a connection between processor and controller.
> * Steinberg::Vst::IMessage: Represents a message to send to the counterpart.
> * Steinberg::Vst::IAttributeList: A list of attributes belonging to a message.
> 
> Please note that messages from the processor to the controller must not be sent during the process call, as this would not be fast enough and would break the real time processing. Such tasks should be handled in a separate timer thread.

## Implementation hints from open source frameworks
### VST3SDK 3.7.8
#### open303

In this example, the processor sends the preset (pattern) to the editor. The other way around should be relatively similar.

[https://github.com/scheffle/open303/blob/850c98186db28829f9ff6c8254966cdb6bf334e6/Source/VST3/o303processor.cpp#L185-L203](https://github.com/scheffle/open303/blob/850c98186db28829f9ff6c8254966cdb6bf334e6/Source/VST3/o303processor.cpp#L185-L203)

```cpp
void sendPatternToController (int patternIndex)
{
	if (!peerConnection)
		return;

	const auto& pattern = open303Core.sequencer.getPattern (patternIndex);
	assert (pattern != nullptr);

	vst3utils::message msg (owned (allocateMessage ()));
	if (!msg.is_valid ())
		return;
	msg.set_id (msgIDPattern);
	auto attributes = msg.get_attributes ();
	if (!attributes.is_valid ())
		return;
	PatternData data = toPatternData (*pattern);
	attributes.set (msgIDPattern, data);
	peerConnection->notify (msg);
}
```

### JUCE 7.0.12
#### Editor part

In JUCE, JuceVST3EditController is the Vst::IEditController

[https://github.com/juce-framework/JUCE/blob/4f43011b96eb0636104cb3e433894cda98243626/modules/juce_audio_plugin_client/juce_audio_plugin_client_VST3.cpp#L4246-L4249](https://github.com/juce-framework/JUCE/blob/4f43011b96eb0636104cb3e433894cda98243626/modules/juce_audio_plugin_client/juce_audio_plugin_client_VST3.cpp#L4246-L4249)

```cpp
return static_cast<Vst::IEditController*> (new JuceVST3EditController (h));
```


Then following Steinberg’s documentation

[https://steinbergmedia.github.io/vst3_dev_portal/pages/Technical+Documentation/API+Documentation/Index.html#private-communication](https://steinbergmedia.github.io/vst3_dev_portal/pages/Technical+Documentation/API+Documentation/Index.html#private-communication)

```cpp
Vst::IEditController* editController = &myJuceVST3EditControllerInstance;
Vst::IConnectionPoint* iConnectionPointController = nullPtr;
editController->queryInterface (Vst::IConnectionPoint::iid, (void**)&iConnectionPointController);
```


Then the sending part should be similar to the 303 example, with peerConnection=editController

[https://github.com/scheffle/open303/blob/850c98186db28829f9ff6c8254966cdb6bf334e6/Source/VST3/o303processor.cpp#L185-L203](https://github.com/scheffle/open303/blob/850c98186db28829f9ff6c8254966cdb6bf334e6/Source/VST3/o303processor.cpp#L185-L203)

```cpp
void sendPatternToController (int patternIndex)
{
	if (!peerConnection)
		return;

	const auto& pattern = open303Core.sequencer.getPattern (patternIndex);
	assert (pattern != nullptr);

	vst3utils::message msg (owned (allocateMessage ()));
	if (!msg.is_valid ())
		return;
	msg.set_id (msgIDPattern);
	auto attributes = msg.get_attributes ();
	if (!attributes.is_valid ())
		return;
	PatternData data = toPatternData (*pattern);
	attributes.set (msgIDPattern, data);
	peerConnection->notify (msg);
}
```

#### Processor Component part

Override in your own JuceVST3Component

[https://github.com/juce-framework/JUCE/blob/4f43011b96eb0636104cb3e433894cda98243626/modules/juce_audio_plugin_client/juce_audio_plugin_client_VST3.cpp#L2542-L2560](https://github.com/juce-framework/JUCE/blob/4f43011b96eb0636104cb3e433894cda98243626/modules/juce_audio_plugin_client/juce_audio_plugin_client_VST3.cpp#L2542-L2560)

```cpp
   tresult PLUGIN_API notify (Vst::IMessage* message) override
```

## What about...
### surrounding bulk performEdit calls with start/finishGroupEdit

The documentation explains that groupEdit are about accurate timestamps, and thus not related to preset browsing.

[https://github.com/steinbergmedia/vst3_pluginterfaces/blob/37de655a51b28a558645621b39d48a00292be5e2/vst/ivsteditcontroller.h#L268-L271](https://github.com/steinbergmedia/vst3_pluginterfaces/blob/37de655a51b28a558645621b39d48a00292be5e2/vst/ivsteditcontroller.h#L268-L271)

```cpp
/** Starts the group editing (call before a \ref IComponentHandler::beginEdit),
the host will keep the current timestamp at this call and will use it for all
\ref IComponentHandler::beginEdit
```

### only calling restartComponent and letting the host transfer changes

A Steinberg developer explains that the host will not transfer changes from editor to component when calling restartComponent.

[https://forums.steinberg.net/t/how-to-use-restartcomponent-and-which-flags-are-the-right-one-when-changing-all-characteristics-parameters-except-size/202031/17](https://forums.steinberg.net/t/how-to-use-restartcomponent-and-which-flags-are-the-right-one-when-changing-all-characteristics-parameters-except-size/202031/17)

> A plug-in calls restartComponent (kParamValuesChanged) to let the host know about an internal change of its parameter values.[…] So the host wont transfer changes when restartComponent (kParamValuesChanged) is called.

