---
title: Logger
description: Instructions on how to enable the logger integration for Open Peer Power.
ha_category:
  - Utility
ha_release: 0.8
ha_quality_scale: internal
ha_codeowners:
  - '@open-peer-power/core'
ha_domain: logger
excerpt: none
---

The `logger` integration lets you define the level of logging activities in Home
Assistant.

To enable the `logger` integration in your installation,
add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
logger:
```

To log all messages and ignore events lower than critical for specified
components:

```yaml
# Example configuration.yaml entry
logger:
  default: info
  logs:
    openpeerpower.components.yamaha: critical
    custom_components.my_integration: critical
```

To ignore all messages lower than critical and log event for specified
components:

```yaml
# Example configuration.yaml entry
logger:
  default: critical
  logs:
    # log level for HA core
    openpeerpower.core: fatal

    # log level for MQTT integration
    openpeerpower.components.mqtt: warning

    # log level for all python scripts
    openpeerpower.components.python_script: warning

    # individual log level for this python script
    openpeerpower.components.python_script.my_new_script.py: debug

    # log level for SmartThings lights
    openpeerpower.components.smartthings.light: info

    # log level for a custom component
    custom_components.my_integration: debug

    # log level for the `aiohttp` Python package
    aiohttp: error

    # log level for both 'glances_api' and 'glances' integration
    openpeerpower.components.glances: fatal
    glances_api: fatal
```

The log entries are in the form  
*timestamp* *log-level* *thread* [**namespace**] *message*  
where **namespace** is the *<component_namespace>* currently logging.

In the example, do note the difference between 'glances_api' and 'openpeerpower.components.glances' namespaces,
both of which are at root. They are logged by different APIs.

If you want to know the namespaces in your own environment then check your log files on startup.
You will see INFO log messages from openpeerpower.loader stating `loaded <component> from <namespace>`.
Those are the namespaces available for you to set a `log level` against.

### Log Levels

Possible log severity levels, listed in order from most severe to least severe, are:

- critical
- fatal
- error
- warning
- warn
- info
- debug
- notset

## Services

### Service `set_default_level`

You can alter the default log level (for integrations without a specified log
level) using the service `logger.set_default_level`.

An example call might look like this:

```yaml
service: logger.set_default_level
data:
  level: info
```

### Service `set_level`

You can alter log level for one or several integrations using the service
`logger.set_level`. It accepts the same format as `logs` in the configuration.

An example call might look like this:

```yaml
service: logger.set_level
data:
  openpeerpower.core: fatal
  openpeerpower.components.mqtt: warning
  openpeerpower.components.smartthings.light: info
  custom_components.my_integration: debug
  aiohttp: error
```

The log information are stored in the
[configuration directory](/docs/configuration/) as `open-peer-power.log`
and you can read it with the command-line tool `cat` or follow it dynamically
with `tail -f`.

You can use the example below, when logged in through the [SSH add-on](/addons/ssh/):

```bash
tail -f /config/open-peer-power.log
```

On Docker you can use your host command line directly - follow the logs dynamically with:

```bash
# follow the log dynamically
docker logs --follow  MY_CONTAINER_ID
```

To see other options use `--help` instead, or simply leave with no options to display the entire log.
