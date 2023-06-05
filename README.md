# blynk_blueprint_asset_tracking
Blynk Asset Tracking Blueprint for tracking assets with [Blynk](https://blynk.io) and a [Particle Boron](https://docs.particle.io/boron/) IoT device.  

## Functional Requirements
- Only publish location information when the GPS has a fix and the location has moved more than 122 m / 400 ft since it was powered on.
- If the position has changed 122 m / 400 ft, set a flag for that event, make it visible to the user, and allow the user to reset it. 
- Track the device location and speed on a map in a web dashboard and mobile app.
- Publish the device position after the hardware boots and a GPS fix is obtained.
- Publish the date/time in UTC when the device last published data.  
- Publish the cellular signal strength and the battery charge status. 

## Overview
A Particle Boron with attached GPS FeatherWing reads the device location. &nbsp; The location data is pushed from the Particle cellular device to the Blynk IoT platform via a Particle Webhook. &nbsp; The data is then visualized on both a Blynk web dashboard and mobile app. &nbsp; A related detailed tutorial [How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk) is available.  

## Hardware
- [Particle Boron](https://docs.particle.io/boron/)
- [GPS FeatherWing](https://www.adafruit.com/product/3133)
- [External active 28 dB GPS antenna](https://www.adafruit.com/product/960) and [a SMA to uFL/u.FL/IPX/IPEX RF adapter cable](https://www.adafruit.com/product/851) are both optional, but highly recommended for the best GPS performance.

The Boron is physically stacked on top of the GPS FeatherWing, completing the electrical connection between them. &nbsp; The Boron and the GPS FeatherWing communicate over the Boron UART pins. 

## Particle Webhook
Create a Particle Webhook to transfer the data from the Particle Boron to Blynk. &nbsp; The article [How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk) describes in detail how to create a Particle webhook. &nbsp; The datastreams in this Blueprint only include what is needed to pass on the location data, so the Particle Webhook configuration that follows is all that is needed. 

Webhook event name:  blynk_https_get<br/><br/>
Full URL:  https://ny3.blynk.cloud/external/api/batch/update<br/><br/>
(update "ny3.blynk.cloud" with your server shown in the Blynk.Console lower right.  See [here](https://docs.blynk.io/en/blynk.cloud/troubleshooting) for a list of valid server addresses).<br/>
Query Parameters:
{
  "token": "{{t}}",
  "V3": "{{lon}},{{lat}}",
  "V4": "{{spd}}",
  "V5": "{{moved}}",
  "V6": "{{PARTICLE_PUBLISHED_AT}}"
}

![alt text](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/f0f8adea0b3feb23fde7f69a0fcef34bb894930d/blynk_blueprint_asset_tracking_particle_webhook(1).png "Particle Webhook")

## Blueprint
Use the 'Blynk Aasset Tracking' Blueprint by navigating in the Blynk.Console to 'Templates -> BLUEPRINTS -> BLUEPRINT -> Use Blueprint'. &nbsp; Choose the 'Activate first device' option. &nbsp; This will generate and show a AuthToken. &nbsp; Copy the AuthToken and keep it in a safe place, and then use it in the next section to update "BLYNK_AUTH_TOKEN" within the sketch  [blynk_blueprint_asset_tracking.ino](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/38192cabe4122f59c3fe6956038b1a33c015e4b6/blynk_blueprint_asset_tracking.ino).

## Firmware
Cellular communication between the hardware and Blynk will utilize the [Blynk HTTPs API](https://docs.blynk.io/en/blynk.cloud/https-api-overview) to minimize cellular data usage. &nbsp; The Blynk library is not used nor needed.  The Particle Boron cellular IoT device will publish a JSON string to the Particle Cloud, referencing a Particle webhook. &nbsp; The webhook reformats the data, and then sends it to the Blynk Cloud via an HTTP GET, updating the Blynk datastreams.  

Open the sketch [blynk_blueprint_asset_tracking.ino](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/38192cabe4122f59c3fe6956038b1a33c015e4b6/blynk_blueprint_asset_tracking.ino) from this repository in [Workbench](https://www.particle.io/workbench/) or other IDE. &nbsp; Install the library "Adafruit_GPS" as noted in the sketch.  

In the sketch, find the code snippet shown below and update "BLYNK_AUTH_TOKEN" with your AuthToken obtained from activating a device from the Blueprint. 

  #define BLYNK_AUTH_TOKEN "your_32_char_token"

Note that the minimum publishing interval (shown below) in the sketch is set to 60000 ms (60 sec) in the sketch and should be updated to a longer duration such as 300000 ms or 5 min after the sketch has been fully tested in order to minimize cellular data usage. 

  const uint32_t TIMER_INTERVAL_MS = 60000;

Save the modified sketch and then upload it to your Particle Boron. &nbsp; Restart your Particle Boron and allow it to connect to the Particle Cloud. 

## Blynk Web Dashboard &amp; Blynk.App Widgets
The **GPS position longitude/latitude coordinates** stored in the datastream V3 can be displayed with a web widget 'Label Display', but not with a Blynk.App widget. &nbsp; The position may be displayed on a web dashboard [map widget](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/addd0f70d18b25faa00ed9e1808185d7083bd6d1/blynk_blueprint_asset_tracking_web_dashboard.png) and a Blynk.App [map widget](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/addd0f70d18b25faa00ed9e1808185d7083bd6d1/blynk_blueprint_asset_tracking_mobile_app(1).png).  

The device **speed in mph** is available in the datastream V4. &nbsp; A variety of web dashboard and Blynk.App widgets can display this value, including Label Display / Value Display / Labeled Value, gauge, chart, etc. &nbsp; The speed may also be added to a web dashboard map widget as an [overlay](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/ea0df8c8faf89e76fb923c34c6448f4ae2f596af/blynk_blueprint_asset_tracking_web_dashboard(4).png), [example](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/3ef24b8f3f158d819880977aad00e9d7b7fd9a81/blynk_blueprint_asset_tracking_web_dashboard(5).png). 

The datastream V5 named '**position_changed**' V5 will be updated to a value of 1 by the hardware when it has changed by more than 122 m / 400 ft since it was powered on, or since the last time data was published (firmware variable TIMER_INTERVAL_MS). &nbsp; The datastream value is not updated to a value of 0 by the hardware, so this should be done with an [automation](https://docs.blynk.io/en/concepts/automations) if the feature is to be used. &nbsp; A variety of widgets including a LED, switch, and value display can be used to visualize the datastream value, and it may also be used in an [automation](https://github.com/markwkiehl/blynk_blueprint_asset_tracking/tree/main#automation) to notify the user of change in value. &nbsp; The hardware determines the change in position from the last published GPS coordinates. &nbsp; The position delta of 122 m / 400 ft can be changed in the hardware, but note that making it smaller can cause false positive triggers if the GPS accuracy is poor.

The last **date/time in [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) the position was last published** by the hardware can be viewed by using a Label Display / Value Display web dashboard / Blynk.App widget. 

## Blynk Web Dashboard
![alt text](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/6bf578d83bda4f1ea99d87ec85d57ee5115d262e/blynk_blueprint_asset_tracking_web_dashboard.png "Web Dashboard")


## Blynk.App
![alt text](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/d7dfaacd1d1f15df0ec53b3d39941914bfe0e007/blynk_blueprint_asset_tracking_mobile_app(1).png "Blynk.App")

## Automation
An [automation](https://docs.blynk.io/en/concepts/automations) can be created to notify the user when the device position has changed more than 122 m / 400 ft since it was powered on, or since the last time data was published (firmware variable TIMER_INTERVAL_MS). &nbsp; See [datastream V5 'position_changed'](https://github.com/markwkiehl/blynk_blueprint_asset_tracking/tree/main#blynk-web-dashboard--blynkapp-widgets) for more details. &nbsp; Details on how to create an automation are in the article [How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk).  

## Troubleshooting
The Particle Console provides information about when the device has last connected, the cellular signal strength, etc.  You can also see what data has been pushed from the device to Blynk by reviewing the Particle Integration webhook log.  

## Related Links
[How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk)<br/>
[How to Control a Particle Device with Blynk](https://blynk.io/blog/how-to-control-particle-device-with-blynk)<br/>
[github.com/blynkkk/blueprints](https://github.com/blynkkk/blueprints)<br/>
