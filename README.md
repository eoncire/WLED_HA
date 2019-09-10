# WLED_HA
WLED basic integration with HomeAssistant

https://imgur.com/lLM5PuZ

This is my way of controlling additional features of WLED via the HomeAssistant frontend (lovelace).  Theres probably much more attractive ways to accomplish this, but this is what I've come up with.  I am by no means a HA / LoveLace / programming veteran of any kind, this is just what I've mustered together to accomplish what I needed.  I don't use the MQTT discovery at all, but might to be able to set a solid color.

**Prerequisites:**

**WLED** - you need WLED flashed to an ESP board of your choice obviously
**HomeAssistant** - I run Hass.io for the ease of add-ons and backups
**MQTT** - You need an MQTT broker set up, using Hass.io there is a simple add on broker
**NodeRed** - Again, using Hass.io there is an easy to install NodeRed add-on

**General Usage / Setup:**

Again, there are probably easier / cleaner ways to accomplish this but this is what I've come up with.

**Input sliders** are created in the config.yaml of HA for the additional things we want to control.  They are Brightness, Effect Speed, Effect Intensity.  That code is in the config.yaml file.  I assume there will be some first time HA / yaml people reading this, don't be scared, but do format it EXACTLY as shown.  Missing a single space / comma / whatever can cuase your config to be invalid, that's a bad thing.  The sliders dont actually control anything themselves, just a dummy slider to be picked up in NodeRed, but they DO reflect changes made if you change the sliders via the WLED webUI or app which is cool.  If you open the webui and adjust the effect speed it will be shown live in HA as well, that was a must have in my opinion.

**Input Booleans** are created to have a nice big button to push to select presets.  Again, probably other ways, I'd love to see ideas.  The presets must be made ahead of time via the WLED webui or app.  I went through and picked a couple that would I thought look good for Christmas lights.  The preset function of WLED is neat, it will save the effect, brightness, speed etc to a memory slot on the ESP board.  They can then be called back up at anytime.  This takes a little work to get set up but I want a nice easy way for my wife to be able to change the effect quickly with a single button.  The input booleans are basically dummy switches that are picked up in NodeRed.





