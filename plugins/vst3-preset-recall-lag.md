---
layout: link
active_page: vst3-preset-lag
title: Lag when changing presets in VST3 plugins
---

# Lag/CPU spikes when changing presets in VST3 plugins

## Introduction

We receive reports from some VST3 plugin developers that their plugin's UIs lag when switching presets in a custom preset browser. CPU spikes may also be observed. The situation is exacerbated when plugins declare many extra parameters for the purpose of handling MIDI CC messages internally. Upon investigation, it seems that some manufacturers call [EditController::performEdit](https://steinbergmedia.github.io/vst3_doc/vstsdk/classSteinberg_1_1Vst_1_1EditController.html#ae49a2edaa0089a662f685f7730b1d288) sequentially for all parameters after they recall a preset. This is done from the VST3 Controller (in the UI) in order to notify both the host and the Processor part (the DSP) of the new parameter states - but this is not a good approach. In our case, it causes a lag as Live needs time to react to `performEdit` calls in its parameter mapping system. We receive these calls synchronously, and cannot optimize for situations where many successive calls happen in a short time frame. Surrounding `performEdit` with calls to VST3's [EditController::start/finishGroupEdit](https://steinbergmedia.github.io/vst3_doc/vstsdk/classSteinberg_1_1Vst_1_1EditController.html#aa6ee73dccd610b996e9ee60abb2401d1) might sound like the solution, but they are meant for grouping timestamped automation events.

## Solution

The solution is to send an [IMessage](https://steinbergmedia.github.io/vst3_doc/vstinterfaces/classSteinberg_1_1Vst_1_1IMessage.html) using [Private Communication](https://steinbergmedia.github.io/vst3_dev_portal/pages/Technical+Documentation/API+Documentation/Index.html#private-communication) from the VST3 Controller to the Processor. The message can contain the values of parameters that need to be updated in the DSP. This approach effectively bypasses Live's parameter mapping system and avoids causing a lag. Once this is done, the Controller should call [IComponentHandler::restartComponent](https://steinbergmedia.github.io/vst3_doc/vstinterfaces/classSteinberg_1_1Vst_1_1IComponentHandler.html#a1f283573728cf0807224c5ebdf3ec3a6) with the argument [kParamValuesChanged](https://steinbergmedia.github.io/vst3_doc/vstinterfaces/namespaceSteinberg_1_1Vst.html#a35f71d02b79cbd11942a149e651373e6a67a358277112f9c5ac816474ae501a09) which notifies the host of the new parameter values. [Here](https://forums.steinberg.net/t/how-to-use-restartcomponent-and-which-flags-are-the-right-one-when-changing-all-characteristics-parameters-except-size/202031/17) a Steinberg engineer explains that a call to `restartComponent(kParamValuesChanged)` alone is not enough to update the Processor/DSP.
## Example code

The following code is an example of how you might implement a custom method in the VST3 Controller that sends an IMessage to the Processor after a preset is recalled in the plugin's UI. The code is just for illustration, it will not compile.

```cpp
class PluginController : public Steinberg::Vst::IEditController
{
  ...
  // This method illustrates what might happen when restoring a preset
  // This preset is for a simple gain plugin... it just sets a single
  // parameter to 0.5
  void recallPreset()
  {
    // First set the controller's representation of the parameter
    setParamNormalized(kParamGainId, 0.5);
    // Then send the IMessage
    sendCustomMessageOnPresetRecall();
    // If the message is not delivered synchronously, it is necessary to send another
    // message back from the processor and restart the component asynchronously 
    // however, in Live it is delivered synchronously.
    Steinberg::FUnknownPtr<Steinberg::Vst::IComponentHandler> handler(componentHandler);
    handler->restartComponent(Steinberg::Vst::RestartFlags::kParamValuesChanged);
  }

private:
  void sendCustomMessageOnPresetRecall()
  {
    OPtr<Steinberg::Vst::IMessage> message = allocateMessage();

    if (message)
    {
      message->setMessageID("PresetRecalled");
      // Here we send a single float, but you could also serialize multiple values
      // for a complex plugin state with a binary message
      message->getAttributes()->setFloat("gainParam", getParamNormalized(kParamGainId));
      
      sendMessage(message);
    }
};

```

And here is how the IMessage is received in the VST3 Processor.

```cpp
class PluginProcessor : public Steinberg::Vst::AudioEffect
{
public:
  ...
  // Audio processing function (called on real-time thread)
  Steinberg::tresult process (Steinberg::Vst::ProcessData& data)
  {    
    if (data.inputParameterChanges)
    {
      // here is where you read regular parameter changes and update
      // mGainParamValue accordingly
    }

    ... 

    float gain = static_cast<float>(mGainParamValue);

    // here is where DSP processing takes place using the gain variable

    return kResultOk;
  }


  // Handle incoming messages (on main/UI thread)
  Steinberg::tresult PLUGIN_API notify (Steinberg::Vst::IMessage* message)
  {
    if (message && std::string_view(message->getMessageID()) == "PresetRecalled")
    {
      Steinberg::Vst::ParamValue paramValue;
      if (message->getAttributes()->getFloat("gainParam", paramValue) == Steinberg::kResultOk)
      {
        mGainParamValue = paramValue;
      }

      return Steinberg::kResultOk;
    }
    
    return Steinberg::kResultFalse;
  }

private:
  std::atomic<Steinberg::Vst::ParamValue> mGainParamValue = 0.0;
};

```

***

[Back](index)
