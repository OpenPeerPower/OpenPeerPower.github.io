---
title: "Customizing entities"
description: "Simple customization for entities in the frontend."
redirect_from: /getting-started/customizing-devices/
excerpt: none
---

## Changing the entity_id

You can use the UI to change the `entity_id` and friendly name of supported entities. To do this:

1. Select the entity, either from the frontend or by clicking <img src='/images/frontend/entity_box.png' /> next to the entity in the Developer Tools "States" tab.
2. Click on the cog in the right corner of the entity's dialog
3. Enter the new name or the new entity ID (remember not to change the domain of the entity - the part before the `.`)
4. Select *Save*

If your entity is not supported, or you cannot customize what you need via this method, please see below for more options:

## Customizing entities

By default, all of your devices will be visible and have a default icon determined by their domain. You can customize the look and feel of your front page by altering some of these parameters. This can be done by overriding attributes of specific entities.

### Customization using the UI

Under the *Configuration* menu you'll find the *Customization* menu. If this menu item is not visible, enable advanced mode on your [profile page](/docs/authentication/#your-account-profile) first. When you select an entity to customize, you'll see all the existing attributes listed and you can customize those or select an additional supported attribute ([see below](/docs/configuration/customizing-devices/#possible-values)). You may also need to add the following to your `configuration.yaml` file, depending when you started using Open Peer Power:

```yaml
openpeerpower:
  customize: !include customize.yaml
```

#### Possible values

#### Device Class

Device class is currently supported by the following components:

* [Binary Sensor](/integrations/binary_sensor/)
* [Sensor](/integrations/sensor/)
* [Cover](/integrations/cover/)
* [Media Player](/integrations/media_player/)

### Manual customization

<div class='note'>

If you implement `customize`, `customize_domain`, or `customize_glob` you must make sure it is done inside of `openpeerpower:` or it will fail.

</div>

```yaml
openpeerpower:
  name: Home
  unit_system: metric
  # etc

  customize:
    # Add an entry for each entity that you want to overwrite.
    sensor.living_room_motion:
      hidden: true
    thermostat.family_room:
      entity_picture: https://example.com/images/nest.jpg
      friendly_name: Nest
    switch.wemo_switch_1:
      friendly_name: Toaster
      entity_picture: /local/toaster.jpg
    switch.wemo_switch_2:
      friendly_name: Kitchen kettle
      icon: mdi:kettle
    switch.rfxtrx_switch:
      assumed_state: false
  # Customize all entities in a domain
  customize_domain:
    light:
      icon: mdi:home
    automation:
      initial_state: 'on'
  # Customize entities matching a pattern
  customize_glob:
    "light.kitchen_*":
      icon: mdi:description
    "scene.month_*_colors":
      hidden: true
      emulated_hue_hidden: false
```

### Reloading customize

 Open Peer Power offers a service to reload the core configuration while Open Peer Power is running called `openpeerpower.reload_core_config`. This allows you to change your customize section and see it being applied without having to restart Open Peer Power. To call this service, go to the "Service" tab under Developer Tools, select the `openpeerpower.reload_core_config` service and click the "CALL SERVICE" button. Alternatively, you can press the "Reload Location & Customizations" button under Configuration > Server Control.

<div class='note warning'>
New customize information will be applied the next time the state of the entity gets updated.
</div>
