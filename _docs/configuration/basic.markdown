---
title: "Setup basic information"
description: "Setting up the basic info of Open Peer Power."
redirect_from: /getting-started/basic/
excerpt: none
---

As part of the default provisioning process, Open Peer Power can detect your location from IP address geolocation. Open Peer Power will automatically select a temperature unit and time zone based on this location. You may adjust this during provisioning, or afterwards at Configuration -> General. 

If you prefer YAML, you can add the following information to your `configuration.yaml`:

```yaml
openpeerpower:
  latitude: 32.87336
  longitude: 117.22743
  elevation: 430
  unit_system: metric
  time_zone: America/Los_Angeles
  name: Home
  whitelist_external_dirs:
    - /usr/var/dumping-ground
    - /tmp
```

### Reload Core Service

 Open Peer Power offers a service to reload the core configuration while Open Peer Power is running called `openpeerpower.reload_core_config`. This allows you to change any of the above sections and see it being applied without having to restart Open Peer Power. To call this service, go to the "Service" tab under Developer Tools, select the `openpeerpower.reload_core_config` service and click the "CALL SERVICE" button. Alternatively, you can press the "Reload Location & Customizations" button under Configuration > Server Control.
