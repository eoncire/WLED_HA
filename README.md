# WLED_HA
WLED basic integration with HomeAssistant

https://imgur.com/lLM5PuZ

This is my way of controlling additional features of WLED via the HomeAssistant frontend (lovelace).  Theres probably much more attractive ways to accomplish this, but this is what I've come up with.  I am by no means a HA / LoveLace / programming veteran of any kind, this is just what I've mustered together to accomplish what I needed.  I don't use the MQTT discovery at all, but might to be able to set a solid color.

## **Prerequisites:**

**WLED** - you need WLED flashed to an ESP board of your choice obviously.

**HomeAssistant** - I run Hass.io for the ease of add-ons and backups.

**MQTT** - You need an MQTT broker set up, using Hass.io there is a simple add on broker.

**NodeRed** - Again, using Hass.io there is an easy to install NodeRed add-on.



## **General Usage / Setup:**

Again, there are probably easier / cleaner ways to accomplish this but this is what I've come up with.

**Input sliders** are created in the config.yaml of HA for the additional things we want to control.  They are Brightness, Effect Speed, Effect Intensity.  That code is in the config.yaml file.  I assume there will be some first time HA / yaml people reading this, don't be scared, but do format it EXACTLY as shown.  Missing a single space / comma / whatever can cuase your config to be invalid, that's a bad thing.  The sliders dont actually control anything themselves, just a dummy slider to be picked up in NodeRed, but they DO reflect changes made if you change the sliders via the WLED webUI or app which is cool.  If you open the webui and adjust the effect speed it will be shown live in HA as well, that was a must have in my opinion.

**Input Booleans** are created to have a nice big button to push to select presets.  Again, probably other ways, I'd love to see ideas.  The presets must be made ahead of time via the WLED webui or app.  I went through and picked a couple that would I thought look good for Christmas lights.  The preset function of WLED is neat, it will save the effect, brightness, speed etc to a memory slot on the ESP board.  They can then be called back up at anytime.  This takes a little work to get set up but I want a nice easy way for my wife to be able to change the effect quickly with a single button.  The input booleans are basically dummy switches that are picked up in NodeRed.

**MQTT** is used for communication from HA to WLED.  You can control EVERYTHING via MQTT.  In the WLED / Sync Interfaces setup you enter the IP of your MQTT server.  If you're using Hass.io it's the IP of your HA server.  The topic can be anything you want, but for simplicity I use wled/d1 as this is my first d1 mini that has WLED.  I plan on having others down the road...  Anyways, if you use an app like MQTT explorer you can look at wled/d1/v which is the topic that WLED posts all of its current settings to upon any change.  If you pick a different color it'll puke out a big long XML blurb with the different settings (SX=speed, FX=effect, IX=intensity).  Again, ANY change to any setting in WLED causes the program to post to this topic with all settings.  That's how we get the effect speed, brightness, and intensity back into the HA input sliders.  The next major MQTT topic is wled/d1/api  That's where we publish to set effect speed, intensity and brightness.  A full list of those options are in the WLED documentation here https://github.com/Aircoookie/WLED/wiki/HTTP-request-API

**NodeRed** is where all of the magic happens really, and it's a mess, but it works.  I'll break it up into sections.  But first a NodeRed overview becuase again I'm sure there will be some NodeRed virgins.  NodeRed has input, action, and output nodes at its most simplest form.  A flow is a page in NR where we drag and drop stuff, you look at the flow from left to right for the most part.  When using the NodeRed add-on for Hass.io it comes with a set of HA specific nodes that will pull data from HA entities (switch status, light vaules, and of course our input sliders and input booleans).  Then we take that data (called a payload in NR) and do something to it like modify its formatting or add additional info into the payload so that WLED can understand it.  Lastly, we output it by publishing a MQTT message to our wled/d1/api topic which WLED is subscribed to.

**Input Slider** data is pulled into NR by an "events: state" node.  Anytime the entity which we pick in that nodes settings is chagned it'll pass it down the string.  Here is my events: state node for the speed.
https://imgur.com/kZRdEHS


