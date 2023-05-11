# blynk_blueprint_asset_tracking
Blynk Asset Tracking Blueprint for tracking assets with [Blynk](https://blynk.io) and a [Particle Boron](https://docs.particle.io/boron/) IoT device.  

## Functional Requirements
- Notify the user if the device has moved more than 122 m / 400 ft since it was powered on.
- Track the device location and speed.

## Hardware
- [Particle Boron](https://docs.particle.io/boron/)
- [GPS FeatherWing](https://www.adafruit.com/product/3133)
## Software
In order to minimize cellular communication between Blynk and the device, the [Blynk HTTPs API[(https://docs.blynk.io/en/blynk.cloud/https-api-overview) will be used.  The Particle Boron cellular IoT device will publish a JSON string to the Particle Cloud, referencing a Particle webhook. The webhook reformats the data, and then sends it to the Blynk Cloud via an HTTP GET, updating the Blynk datastreams.  

Begin by using the 'Blynk Aasset Tracking' Blueprint by navigating in the Blynk.Console to 'Templates -> BLUEPRINTS -> BLUEPRINT -> Use Blueprint'.  

The article [How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk) describes in detail how to do all of that. 

## Blynk


## Related Links
[How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk)
[github.com/blynkkk/blueprints](https://github.com/blynkkk/blueprints)
