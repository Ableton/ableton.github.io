---
layout: link
active_page: third_party_plugin_dev_portal
title: Third party plugin developers portal
---

# Third party plugin developers portal

## Lag or CPU spikes when changing presets in VST3 plugins

When a VST3 plugin editor needs to change many parameter values at once, for
example when changing presets, it should not call `performEdit` for each
parameter.


Instead, the editor should use private communication (for example sending
custom `IMessage`) to send all the new values to the processor. Then, the
processor shall be able to return the correct information before making a call
to `restartComponent(kParamValuesChanged)`.

More details at [performEdit alternatives](VST3presetSwitchingPerformEditAlternatives).

