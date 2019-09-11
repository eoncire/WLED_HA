# WLED_HA
WLED basic integration with HomeAssistant

https://imgur.com/lLM5PuZ

Quick overview video ->  https://youtu.be/hwi5DyO02x0

This is my way of controlling additional features of WLED via the HomeAssistant frontend (lovelace).  Theres probably much more attractive ways to accomplish this, but this is what I've come up with.  I am by no means a HA / LoveLace / programming veteran of any kind, this is just what I've mustered together to accomplish what I needed.  I don't use the MQTT discovery at all, but might to be able to set a solid color.

# **Prerequisites:**

**WLED** - you need WLED flashed to an ESP board of your choice obviously.

**HomeAssistant** - I run Hass.io for the ease of add-ons and backups.

**MQTT** - You need an MQTT broker set up, using Hass.io there is a simple add on broker.

**NodeRed** - Again, using Hass.io there is an easy to install NodeRed add-on.  You have to add you MQTT broker info to NodeRed.



# **General Usage / Setup:**

Again, there are probably easier / cleaner ways to accomplish this but this is what I've come up with.

### Input Sliders

**Input sliders** are created in the config.yaml of HA for the additional things we want to control.  They are Brightness, Effect Speed, Effect Intensity.  That code is in the config.yaml file.  I assume there will be some first time HA / yaml people reading this, don't be scared, but do format it EXACTLY as shown.  Missing a single space / comma / whatever can cuase your config to be invalid, that's a bad thing.  The sliders dont actually control anything themselves, just a dummy slider to be picked up in NodeRed, but they DO reflect changes made if you change the sliders via the WLED webUI or app which is cool.  If you open the webui and adjust the effect speed it will be shown live in HA as well, that was a must have in my opinion.

### Input Booleans

 **Input Booleans** are created to have a nice big button to push to select presets.  Again, probably other ways, I'd love to see ideas.  The presets must be made ahead of time via the WLED webui or app.  I went through and picked a couple that would I thought look good for Christmas lights.  The preset function of WLED is neat, it will save the effect, brightness, speed etc to a memory slot on the ESP board.  They can then be called back up at anytime.  This takes a little work to get set up but I want a nice easy way for my wife to be able to change the effect quickly with a single button.  The input booleans are basically dummy switches that are picked up in NodeRed.

### MQTT

**MQTT** is used for communication from HA to WLED.  You can control EVERYTHING via MQTT.  In the WLED / Sync Interfaces setup you enter the IP of your MQTT server.  If you're using Hass.io it's the IP of your HA server.  The topic can be anything you want, but for simplicity I use wled/d1 as this is my first d1 mini that has WLED.  I plan on having others down the road...  Anyways, if you use an app like MQTT explorer you can look at wled/d1/v which is the topic that WLED posts all of its current settings to upon any change.  If you pick a different color it'll puke out a big long XML blurb with the different settings (SX=speed, FX=effect, IX=intensity).  Again, ANY change to any setting in WLED causes the program to post to this topic with all settings.  That's how we get the effect speed, brightness, and intensity back into the HA input sliders.  The next major MQTT topic is wled/d1/api  That's where we publish to set effect speed, intensity and brightness.  A full list of those options are in the WLED documentation here https://github.com/Aircoookie/WLED/wiki/HTTP-request-API


### NodeRed

**NodeRed** is where all of the magic happens really, and it's a mess, but it works.  After you import the flow to your NodeRed add-on you have to make a couple of changes so NodeRed connects to your MQTT broker via its IP address and change the device topic in two of the MQTT nodes so that it reads and posts to the device topic you set up in WLED.  I'll break it up into sections.  But first a NodeRed overview becuase again I'm sure there will be some NodeRed virgins.  NodeRed has input, action, and output nodes at its most simplest form.  A flow is a page in NR where we drag and drop stuff, you look at the flow from left to right for the most part.  When using the NodeRed add-on for Hass.io it comes with a set of HA specific nodes that will pull data from HA entities (switch status, light vaules, and of course our input sliders and input booleans).  Then we take that data (called a payload in NR) and do something to it like modify its formatting or add additional info into the payload so that WLED can understand it.  Lastly, we output it by publishing a MQTT message to our wled/d1/api topic which WLED is subscribed to.

First step is to add your MQTT broker info to NodeRed so it can read and publish to the broker.  After you import the flow open either the **MQTT in** or **MQTT out** node.  Click the pencil icon next to the server name.  Enter the IP of your MQTT broker.  I have my MQTT broker set to allow anomymous connections so I don't use the security tab, that is up to you and your personal configuration.  The same must be done to ensure NodeRed is connected to HomeAssistant.  Open any of the event: state nodes and click on the edit pencil icon next to the server name.  I use Hass.io so i just check the box that says "I use Hass.io".  Those nodes should say "Connected" or have a little green box undreneath them or it wont work.

https://imgur.com/o6pEbE2   https://imgur.com/W4ynViB


**Input Slider** data is pulled into NR by an "events: state" node.  Anytime the entity which we pick in that nodes settings is chagned it'll pass it down the string.  Here is my events: state node for the speed.

https://imgur.com/kZRdEHS

Again if you're using the NR add-on for Hass.io it should automatically connect to your HA server and when you start to type in the name of the entity it should autopopulate and search by entity name.  I called this input slider ledspeed in my config.yaml.  Now any time that slider is changed in the HA frontend it will pass that data along in this flow.  What we want to do with that data is convert it to the proper formatting so that we can tell WLED to adjust the effect speed.  Looking at the HTTP API documentation it says that is done by calling SX=123.  What we need to do is use a **Template** node to modify the payload.  Remember, the payload is just the NR term for the data we're working with.  The data from our input_slider is just a number.  We need it to be SX=thatnumber.  Using a template node like the image below will change the template of the data from a number to mustache template.

https://imgur.com/dZvjMNF

This will take our current payload of a number and insert it in the code where it says payload in orange.  The payload after this node will then be SX=123 That can be connected to a debug node to double check that everything is correct. Debug node is your friend.

Lastly we want to send that speed data payload which is now properly formatted to WLED to actually change the speed.  Pick up a **MQTT Out** node and drop it in the flow with a topic of wled/d1/api  Connect the Template node to it, Deploy changes and it should be set up.  

That same outline is done for brightness as well as effect intensity.  Dummy slider in config with a name of whatver you want, event state node to get that slider data, template node to change it to the CORRECT API syntax, MQTT Out to publish.

This is what that section of my flow looks like.  Event state (the input) node on the left, template node doing it's magic in the middle, MQTT out (the output) node on the right, along with the super helpful debug node.

https://imgur.com/7RkPNZK

**Input Boolean**

**Input Booleans** are used as a dummy button to pick a preset which is saved in WLED.  Note here, if you have to set up a new WLED board in the event of frying one or whatever you WILL need to recreate the presets in WLED.  I don't have a way around that yet, maybe Aircookie can have an export preset option in the future, wink wink.  We use the same event: state node to pull in the data.  This is a portion of my code where there's probably a much more streamline way to do it but this works and I'm not that smart.  An event: state node is inserted for every preset / input_boolean we want to control / use.  I just made 4 nice presets for this proof of concept, TwinkleC9 (Twinklefox effect w/ C9 color pallete), NoiseC9 (Noise3 w/ C9 color), Pride (Pride2015 effect), and XmasPuke (Xmas effect).  My input_booleans are just called preset_1, preset_2, etc in my config.yaml.  You can name them whatever you want as long as it's unique to HA.  NodeRed reads the state of the input boolean, and upon ANY change (important) it'll send a payload down the line.  We use the template node again to format our payload to moustache template, and set the proper code with it.  For this we reference the HTTP API section in the documentation again (nice docs Aircookie, much more organized than this wall of text), and see that presets are called with PL=ourpresetnumber.  I did 4 presets for this demo, so i have 4 nodes set up in this flow.  Each template node calls a different number, PL=1, PL=2, and so on.

https://imgur.com/XPxgfOC

Here is an overview of the flow.  The input slider stuff at the top, and preset stuff at the bottom.  The "2" and "4" nodes there are another magical node call **Inject** node.  They allow you to, wait for it, inject a payload into the flow.  I was testing this setup before I had the HA side figured out and needed to send the number 2 and 4 down the line to make sure I could have WLED call presets via MQTT.  Spoiler alert, it worked.

https://imgur.com/XoRm4CW

Lastly but certainly not leastly is making the sliders in HA reflect changes to the settings done elsewhere like the webui or app for WLED.  We use the **MQTT in** node to pull all of the data that WLED pukes out to wled/d1/v into our flow.  We only need a couple of the bits of the data so we have to cherry pick it.  That is done with the XML function node.  I'm going to get this wrong I'm sure becuase again, I'm not that smart, but the data posted to the wled/d1/v topic is formatted in javascript i think.  We want it in XML so we can pick through it, this node does just that.  Then we take the XML payload which we can make sense of and is organized and pick out the variables we want using the **Change** node as shown below.

https://imgur.com/Fc7r0ZU

We look through the nicely formatted XML data for the line that says payload.vs.sx.[0].  That is what WLED published as it's current effect speed.  We're just stripping the extra info out so we have only a number as our payload here, sort of the opposite of what we did with the template node earlier.  Now we can connect that node to a call service node with our input_slider that we are using for effect speed.  Again, any changes made to WLED and it'll publish ALL OF THE SETTINGS to wled/d1/v.  We cant make sense of the javascript so we change it to XML formatting, then we look for the line that is has our speed number in it, then we strip all of the BS out and are left with just a number.  NodeRed then calls a HA service to set that slider to that value.  Like i said at the top, it aint pretty, but it works.

https://imgur.com/0BEtKjH

And here is the overview for the part of the flow that handles this.  MQTT input node on the left getting the javascript data that WLED posted, XML node next to change formatting so we can make sense of it, then change node to pull out the data we want, then HA call to set the slider to that data.

https://imgur.com/M3WrHGG



### HomeAssistant

You have to add the input_sliders and input_booleans to your config.yaml.  You can copy / paste that code if you would like or just add to existing.  That's pretty straight forward if you have any HA experience.  The config in lovelace is pretty straight forward for those.  I have an entity card with the sliders.  The push buttons for effects are an Entity Button card, single item per card.  You just pick the input_boolean number you want and make sure to use the correct one when setting up NR.  That's really the easiest part.  Again, probably a better way, blah blah blah but this works.

Out of the box if you followed this guide you will have input sliders and some buttons in your "Unused Entities" in HomeAssistant.  BUUUUUT, those buttons wont do anything.  All they do is tell WLED to apply a preset number (1-4).  YOU have to go into WLED and make your own presets.  I cant do that for you, although I could hard-code some into NodeRed by having it send an MQTT message w/ FX, SX, SI and BR values but that sounds like a lot of work....

Enjoy!  Find me on the WLED discord server @e_town, send me a DM or mention if you're having issues.







