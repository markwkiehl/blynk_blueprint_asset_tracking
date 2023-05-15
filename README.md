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

## Particle Webhook
Create a Particle Webhook to transfer the data from the Particle Boron to Blynk.  The article [How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk) describes in detail how to create a Particle webhook. The datastreams in this Blueprint only include what is needed to pass on the location data, so the Particle Webhook configuration that follows is all that is needed. 

Webhook event name:  blynk_https_get<br/>
Full URL:  https://ny3.blynk.cloud/external/api/batch/update    (update "ny3.blynk.cloud" with your server shown in the Blynk.Console lower right.  See [here](https://docs.blynk.io/en/blynk.cloud/troubleshooting) for a list of valid server addresses).<br/>
Query Parameters:
{
  "token": "{{t}}",
  "V3": "{{lon}},{{lat}}",
  "V4": "{{spd}}",
  "V5": "{{moved}}",
  "V6": "{{PARTICLE_PUBLISHED_AT}}"
}

<img src="https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/f0f8adea0b3feb23fde7f69a0fcef34bb894930d/blynk_blueprint_asset_tracking_particle_webhook(1).png"  width="700" border="5"/>

## Software
In order to minimize cellular communication between Blynk and the device, the [Blynk HTTPs API](https://docs.blynk.io/en/blynk.cloud/https-api-overview) will be used.  The Particle Boron cellular IoT device will publish a JSON string to the Particle Cloud, referencing a Particle webhook. The webhook reformats the data, and then sends it to the Blynk Cloud via an HTTP GET, updating the Blynk datastreams.  

Begin by using the 'Blynk Aasset Tracking' Blueprint by navigating in the Blynk.Console to 'Templates -> BLUEPRINTS -> BLUEPRINT -> Use Blueprint'.  Choose the 'Activate first device' option.  This will generate and show a AuthToken.  Copy the AuthToken and keep it in a safe place, and then use it in the next section to update "BLYNK_AUTH_TOKEN" within the sketch  [blynk_blueprint_asset_tracking.ino](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/38192cabe4122f59c3fe6956038b1a33c015e4b6/blynk_blueprint_asset_tracking.ino).


## Firmware
Open the sketch [blynk_blueprint_asset_tracking.ino](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/38192cabe4122f59c3fe6956038b1a33c015e4b6/blynk_blueprint_asset_tracking.ino) from this repository in [Workbench](https://www.particle.io/workbench/) or other IDE.  Install the library "Adafruit_GPS" as noted in the sketch.  

In the sketch, find the code snippet shown below and update "BLYNK_AUTH_TOKEN" with your AuthToken obtained from activating a device from the Blueprint. 

  #define BLYNK_AUTH_TOKEN "your_32_char_token"

Note that the minimum publishing interval (shown below) in the sketch is set to 60000 ms (60 sec) in the sketch and should be updated to a longer duration such as 300000 ms or 5 min after the sketch has been fully tested in order to minimize cellular data usage. 

  const uint32_t TIMER_INTERVAL_MS = 60000;

Save the modified sketch and then upload it to your Particle Boron.  

## Related Links
[How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk)<br/>
[github.com/blynkkk/blueprints](https://github.com/blynkkk/blueprints)
