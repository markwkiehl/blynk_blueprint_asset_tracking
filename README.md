# blynk_blueprint_asset_tracking
Blynk Asset Tracking Blueprint for tracking assets with [Blynk](https://blynk.io) and a [Particle Boron](https://docs.particle.io/boron/) IoT device.  

## Functional Requirements
- Only publish location information when the GPS has a fix and the location has moved more than 122 m / 400 ft since it was powered on.
- Track the device location and speed on a map in a web dashboard and mobile app.

## Hardware
- [Particle Boron](https://docs.particle.io/boron/)
- [GPS FeatherWing](https://www.adafruit.com/product/3133)
- [External active 28 dB GPS antenna](https://www.adafruit.com/product/960) and [a SMA to uFL/u.FL/IPX/IPEX RF adapter cable](https://www.adafruit.com/product/851) are both optional, but highly recommended for the best GPS performance.

The Boron and the GPS FeatherWing communicate over the Boron UART pins. Data is pushed from the Particle cellular device to the Blynk IoT platform via a Particle Webhook.  The data is then visualized on both a Blynk web dashboard and mobile app. A related detailed tutorial [How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk) is available.  

## Software
In order to minimize cellular communication between Blynk and the device, the [Blynk HTTPs API](https://docs.blynk.io/en/blynk.cloud/https-api-overview) will be used.  The Particle Boron cellular IoT device will publish a JSON string to the Particle Cloud, referencing a Particle webhook. The webhook reformats the data, and then sends it to the Blynk Cloud via an HTTP GET, updating the Blynk datastreams.  

Begin by using the 'Blynk Aasset Tracking' Blueprint by navigating in the Blynk.Console to 'Templates -> BLUEPRINTS -> BLUEPRINT -> Use Blueprint'.  



## Particle Webhook
The article [How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk) describes in detail how to create a Particle webhook. The datastreams in this Blueprint only include what is needed to pass on the location data, so the Particle Webhook configuration that follows is all that is needed. 

Webhook event name:  blynk_https_get
Full URL:  https://ny3.blynk.cloud/external/api/batch/update    (update "ny3.blynk.cloud" with your server shown in the Blynk.Console lower right.  See [here](https://docs.blynk.io/en/blynk.cloud/troubleshooting) for a list of valid server addresses).  

<img src="https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/f0f8adea0b3feb23fde7f69a0fcef34bb894930d/blynk_blueprint_asset_tracking_particle_webhook(1).png"  width="100" height="100" border="10"/>

## Blynk


## Related Links
[How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk)
[github.com/blynkkk/blueprints](https://github.com/blynkkk/blueprints)
