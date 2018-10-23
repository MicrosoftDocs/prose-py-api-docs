---
title: Configuring the Microsoft PROSE Code Accelerator SDK - Python
ms.date: 10/22/2018
ms.topic: conceptual
ms.service: non-product-specific
author: simmdan
ms.author: dsimmons
description: Learn how to configure settings for the PROSE Code Accelerator for Python.

---

# Configuring the Microsoft PROSE Code Accelerator SDK

For most cases PROSE Code Accelerator has reasonable configuration defaults, but there are a few scenarios where you
might want to override those default settings.  This is done with the `prose.codeaccelerator.config` variable which is a
dictionary indexed by setting name.  You can examine or change the current value of any setting and future operations in
the current python process will use the new setting.

## Usage

``` python
import prose.codeaccelerator as cx

cx.config["default_target"] = "pyspark"
```

## Configuration Settings

User configurable settings include:

- `default_target` : This sets the default code generation target value for newly created builders.  Most builders set
  their target value to `"pandas"` by default, but if, for instance, you would like to change the target to `"pyspark"`
  for all future builders created in this session, you can change this setting.

- `ipython_display` : If this is set to the value `"repr"`, then even when Code Accelerator is used in an environment
  that calls `ipython_display` (like a Jupyter notebook), it only renders the minimal output used for text-based
  environments.  Otherwise, when in a notebook environment, Code Accelerator will return HTML-based output for a richer
  user experience.

- `telemetry_opt_out` : Set to `False` (default) instructs PROSE to send telemetry information for the purpose of
  improving the product.  The data collected is not used to identify any person. Set to `True` instructs PROSE not to
  send telemetry information.  NOTE: You may also set the environment variable `PROSE_TELEMETRY_OPTOUT` to any value,
  and it's presence will cause this setting to be set to `True` and prevent telemetry information from being sent.

## Persisting configuraiton changes

If you want to change a setting for all future python processes, you will need to create/modify the PROSE configuration
file found at `~/.config/prose/config.json`.  That file is read by PROSE when the Code Accelerator namespace is
imported, so changing the file will not affect your current process--only future processes.

### Example config.json file

``` json
{
    "default_target": "pyspark",
    "ipython_display": "repr"
}
```