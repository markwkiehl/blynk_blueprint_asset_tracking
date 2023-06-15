# blynk_blueprint_asset_tracking
Blynk Asset Tracking Blueprint for tracking assets with [Blynk](https://blynk.io) and a [Particle Boron](https://docs.particle.io/boron/) IoT device.  

## Functional Requirements
- Publish the cellular signal strength, signal quality, and the battery charge status as soon as possible after the device starts (boot), and thereafter along with the position information. 
- Only publish location information when the GPS has a fix, and the specified publish interval TIMER_INTERVAL_MS (5 min default) has elapsed.
- If the position has changed 122 m / 400 ft, set a flag for that event, make it visible to the user, and allow the user to reset it. 
- Track the device location and speed on a map in a web dashboard and mobile app.
- Publish the device position after the hardware boots and a GPS fix is obtained.
- Publish the date/time in UTC when the device last published data.  

## Overview
A Particle Boron with attached GPS FeatherWing reads the device location. &nbsp; The location data is pushed from the Particle cellular device to the Blynk IoT platform via a Particle Webhook. &nbsp; The data is then visualized on both a Blynk web dashboard and mobile app. &nbsp; A related detailed tutorial [How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk) is available.  

## Hardware
- [Particle Boron](https://docs.particle.io/boron/)
- [GPS FeatherWing](https://www.adafruit.com/product/3133)
- [External active 28 dB GPS antenna](https://www.adafruit.com/product/960) and [a SMA to uFL/u.FL/IPX/IPEX RF adapter cable](https://www.adafruit.com/product/851) are both optional, but highly recommended for the best GPS performance.

The Boron is physically stacked on top of the GPS FeatherWing, completing the electrical connection between them. &nbsp; The Boron and the GPS FeatherWing communicate over the Boron UART pins. 

## Particle Webhook
Create a Particle Webhook to transfer the data from the Particle Boron to Blynk. &nbsp; 
Two webhooks are required, one with 'event name' of 'blynk_https_get_boot' is called when the hardware initially starts and only publishes the cellular connection and battery status information. &nbsp;  The second webhook with 'event name' of 'blynk_https_get' is called every TIMER_INTERVAL_MS (5 min default) and includes all of the data.  

The article [How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk) describes in detail how to create a Particle webhook. &nbsp; The datastreams in this Blueprint only include what is needed to pass on the location data, so the Particle Webhook configuration that follows is all that is needed. 

### Webhook event name:  **blynk_https_get_boot**<br/><br/>
Full URL:  https://ny3.blynk.cloud/external/api/batch/update<br/><br/>
(update "ny3.blynk.cloud" with your server shown in the Blynk.Console lower right.  See [here](https://docs.blynk.io/en/blynk.cloud/troubleshooting) for a list of valid server addresses).<br/>
Query Parameters:
{
  "token": "{{t}}",
  "V6": "{{PARTICLE_PUBLISHED_AT}}",
  "V10": "{{v10}}",
  "V11": "{{v11}}",
  "V12": "{{v12}}"
}

![alt text](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/f7dcb57437c696e63607d6f611f631cc4b67dc35/blynk_blueprint_asset_tracking_particle_webhook_No2(1).png "Particle Webhook #1")


### Webhook event name:  **blynk_https_get**<br/><br/>
Full URL:  https://ny3.blynk.cloud/external/api/batch/update<br/><br/>
(update "ny3.blynk.cloud" with your server shown in the Blynk.Console lower right.  See [here](https://docs.blynk.io/en/blynk.cloud/troubleshooting) for a list of valid server addresses).<br/>
Query Parameters:
{
  "token": "{{t}}",
  "V3": "{{lon}},{{lat}}",
  "V4": "{{spd}}",
  "V5": "{{moved}}",
  "V6": "{{PARTICLE_PUBLISHED_AT}}",
  "V10": "{{v10}}",
  "V11": "{{v11}}",
  "V12": "{{v12}}"
}

![alt text](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/1b207282e125cf8f9209113834738008715f28d7/blynk_blueprint_asset_tracking_particle_webhook.png "Particle Webhook #2")

## Blueprint
Use the 'Blynk Aasset Tracking' Blueprint by navigating in the Blynk.Console to 'Templates -> BLUEPRINTS -> BLUEPRINT -> Use Blueprint'. &nbsp; Choose the 'Activate first device' option. &nbsp; This will generate and show a AuthToken. &nbsp; Copy the AuthToken and keep it in a safe place, and then use it in the next section to update "BLYNK_AUTH_TOKEN" within the sketch  [blynk_blueprint_asset_tracking.ino](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/705f4a11bb6a908dc02d608ab2219f77a8baf27c/blynk_blueprint_asset_tracking.ino).

## Firmware
Cellular communication between the hardware and Blynk will utilize the [Blynk HTTPs API](https://docs.blynk.io/en/blynk.cloud/https-api-overview) to minimize cellular data usage. &nbsp; The Blynk library is not used nor needed.  The Particle Boron cellular IoT device will publish a JSON string to the Particle Cloud, referencing a Particle webhook. &nbsp; The webhook reformats the data, and then sends it to the Blynk Cloud via an HTTP GET, updating the Blynk datastreams.  

Open the sketch [blynk_blueprint_asset_tracking.ino](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/705f4a11bb6a908dc02d608ab2219f77a8baf27c/blynk_blueprint_asset_tracking.ino) from this repository in [Workbench](https://www.particle.io/workbench/) or other IDE. &nbsp; Install the library "Adafruit_GPS" as noted in the sketch.  

In the sketch, find the code snippet shown below and update "BLYNK_AUTH_TOKEN" with your AuthToken obtained from activating a device from the Blueprint. 

  #define BLYNK_AUTH_TOKEN "your_32_char_token"

Note that the minimum publishing interval (shown below) in the sketch is set to 60000 ms (60 sec) in the sketch and should be updated to a longer duration such as 300000 ms or 5 min after the sketch has been fully tested in order to minimize cellular data usage. 

  const uint32_t TIMER_INTERVAL_MS = 60000;

Save the modified sketch and then upload it to your Particle Boron. &nbsp; Restart your Particle Boron and allow it to connect to the Particle Cloud. 

## Blynk Web Dashboard &amp; Blynk.App Widgets
The **GPS position longitude/latitude coordinates** stored in the datastream V3 can be displayed with a web widget 'Label Display', but not with a Blynk.App widget. &nbsp; The position may be displayed on a web dashboard [map widget](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/c9bf8a57457cfbf7fdd5697e437be6ecae2f436f/blynk_blueprint_asset_tracking_web_dashboard.png) and a Blynk.App [map widget](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/c9bf8a57457cfbf7fdd5697e437be6ecae2f436f/blynk_blueprint_asset_tracking_mobile_app(2).png).  

The device **speed in mph** is available in the datastream V4. &nbsp; A variety of web dashboard and Blynk.App widgets can display this value, including Label Display / Value Display / Labeled Value, gauge, chart, etc. &nbsp; The speed may also be added to a web dashboard map widget as an [overlay](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/ea0df8c8faf89e76fb923c34c6448f4ae2f596af/blynk_blueprint_asset_tracking_web_dashboard(4).png), [example](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/3ef24b8f3f158d819880977aad00e9d7b7fd9a81/blynk_blueprint_asset_tracking_web_dashboard(5).png). 

The datastream V5 named '**position_changed**' will be updated to a value of 1 by the hardware when it has changed by more than 122 m / 400 ft since it was powered on, or since the last time data was published (firmware variable TIMER_INTERVAL_MS). &nbsp; The datastream value is not updated to a value of 0 by the hardware, so this should be done with an [automation](https://docs.blynk.io/en/concepts/automations) if the feature is to be used. &nbsp; A variety of widgets including a LED, switch, and value display can be used to visualize the datastream value, and it may also be used in an [automation](https://github.com/markwkiehl/blynk_blueprint_asset_tracking/tree/main#automation) to notify the user of change in value. &nbsp; The hardware determines the change in position from the last published GPS coordinates. &nbsp; The position delta of 122 m / 400 ft can be changed in the hardware, but note that making it smaller can cause false positive triggers if the GPS accuracy is poor.

The datastream V10 named '**batt_charge**' is the battery charge.  It will show "no battery" when no battery is connected, otherwise 0.0 to 100.0 where a larger value is better. 

The datastream V11 named '**cell_strength**' is the cellular connection strength.  It is "N/A" when the value is unknown, otherwise 0.0 to 100.0 where a larger value is better. 

The datastream V12 named '**cell_quality**' is the cellular connection quality.  It is "N/A" when the value is unknown, otherwise 0.0 to 100.0 where a larger value is better. 

The last **date/time in [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) the position was last published** by the hardware can be viewed by using a Label Display / Value Display web dashboard / Blynk.App widget. 

## Blynk Web Dashboard
![alt text](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/e49d364d912b49fbd8072db8d5a6ab1d69a2c278/blynk_blueprint_asset_tracking_web_dashboard.png "Web Dashboard")


## Blynk.App
![alt text](https://raw.githubusercontent.com/markwkiehl/blynk_blueprint_asset_tracking/b50557c68d24fac4bd62cea8ab195dd4d4be56a9/blynk_blueprint_asset_tracking_mobile_app(2).png "Blynk.App")

## Automation
An [automation](https://docs.blynk.io/en/concepts/automations) can be created to notify the user when the device position has changed more than 122 m / 400 ft since it was powered on, or since the last time data was published (firmware variable TIMER_INTERVAL_MS). &nbsp; See [datastream V5 'position_changed'](https://github.com/markwkiehl/blynk_blueprint_asset_tracking/tree/main#blynk-web-dashboard--blynkapp-widgets) for more details. &nbsp; Details on how to create an automation are in the article [How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk).  

## Troubleshooting
After the device has started (boot), approximately 10 seconds after it connects to the Particle cloud the cellular connection status is acquired and that along with the battery charge is transmitted. 
Every TIMER_INTERVAL_MS (5 min default) the device will check if a GPS fix is obtained, and if the GPS has a fix then all of the data will be published. This is implemented to prevent the sending of incorrect position values, and to prevent a false positive for position_changed (datastream V5).  
The Particle Console provides information about when the device has last connected, the cellular signal strength, etc.  You can also see what data has been pushed from the device to Blynk by reviewing the Particle Integration webhook log.  

## Related Links
[How to connect a Particle device to Blynk](https://blynk.io/blog/how-to-connect-a-particle-device-to-blynk)<br/>
[How to Control a Particle Device with Blynk](https://blynk.io/blog/how-to-control-particle-device-with-blynk)<br/>
[github.com/blynkkk/blueprints](https://github.com/blynkkk/blueprints)<br/>
