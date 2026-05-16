---
concept: External Logic Model
last_compiled: 2026-05-16
topics_connected: [apps-system, api-reference, mqtt-integration, project-overview]
status: active
---

# External Logic Model

## Pattern
AWTRIX 3 deliberately keeps all application logic outside the device. Custom apps are "dynamic pages" that display content sent from external systems -- they do not store or execute their own logic. The firmware is a rendering engine and communication interface, not an application platform. All intelligence (what to display, when to update, data processing) lives in the smart home system, Node-RED flow, or other external controller.

This pattern extends to the MQTT placeholder feature (`{{topic/path}}`), which allows subscribing to external MQTT topics and displaying their payload without any intermediate processing.

## Instances
- **Custom apps** in [apps-system](../topics/apps-system.md): "In AWTRIX, CustomApps function more like dynamic pages that rotate within the Apploop rotation of the display. These pages do not store or execute their own logic."
- **MQTT placeholders** in [mqtt-integration](../topics/mqtt-integration.md): `{"text": "Solar: {{inverter/total/P_AC}} W"}` pulls live MQTT data directly into display text
- **Node-RED example** in [apps-system](../topics/apps-system.md): YouTube subscriber count flow demonstrates the full pattern -- external system fetches data, formats JSON, pushes to AWTRIX
- **AWTRIX Flows** in [apps-system](../topics/apps-system.md): community-shared automations at flows.blueforcer.de, all running externally

## What This Means
This architecture is a deliberate product decision that trades on-device capability for flexibility and longevity. Benefits: no firmware updates needed for new "apps", flash memory preserved for icons and melodies, no API lock-in (any system that speaks MQTT or HTTP works). The cost is that AWTRIX requires an always-connected external system to show anything beyond the built-in time/date/temp/humidity/battery apps. The device is fundamentally a smart display, not a smart clock.

## Sources
- [apps-system](../topics/apps-system.md)
- [api-reference](../topics/api-reference.md)
- [mqtt-integration](../topics/mqtt-integration.md)
- [project-overview](../topics/project-overview.md)
