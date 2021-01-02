---
title: "MQTT Birth and Last will"
description: "Instructions on how to setup MQTT birth and last will messages within Open Peer Power."
logo: mqtt.png
excerpt: none
---

MQTT supports so-called Birth and Last Will and Testament (LWT) messages. The former is used to send a message after the service has started, and the latter is used to notify other clients about an ungracefully disconnected client.

To integrate MQTT Birth and Last Will messages into Open Peer Power, add the following section to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
mqtt:
  birth_message:
    topic: 'hass/status'
    payload: 'online'
  will_message:
    topic: 'hass/status'
    payload: 'offline'
```
