# openhab-nibe-heatpump

## Summary
openHAB had several bindings for a NIBE heatpump but a widget to show the status on your dashboard is missing. That's the reason for this project. In addition to show the status it has a rule to derive information from the raw data. For instance what's is it doing in terms of nothing, heating, passive heating, cooling or heating your boiler. To help to save energy it calculates a running 3 and 24 hours average on minutes in each mode and estimated energy consumption. This enables experiments since it's possible to see if a change increases or decreases heating minutes.

![example 1](images-wiki/nb-o.png?raw=true)
![example 1](images-wiki/nb-h.png?raw=true)
![example 1](images-wiki/nb-p.png?raw=true)

left to right, doing nothing, heating and passive heating

![example 1](images-wiki/nb-c.png?raw=true)
![example 1](images-wiki/nb-b.png?raw=true)
![example 1](images-wiki/nb-pop.png?raw=true)

left to right, cooling, boiler and info popup

Features
1. It's based on the little NIBE house that is also present in your thermostat. The window shows the current status. 
	* A pause button, it's off
	* A filled flame, it's active heating
	* An open flame, it's passive heating (brine is off, but it's spreading heat)
	* A snowflake, it's cooling
	* A waterdrop, it's heating water in the boiler
2. The outdoor temperature is shown in the left upper corner.
	* Pressing it will show a 24 hours temperature graph.
2. The indoor temperature is shown in the middle of the house. 
3. The (estimated) ground temperature is shown in the middle bottom
4. It shows the heat exchangers for the room, brine and boiler.
	* Pump speed and temperature in & out are shown
	* When off it's dark green
	* When running the hot side is red and the cold side is blue.
	* Of course, depending on cooling or heating cold/hot side will be flipped
	* Pressing the pump speed will show an pop-up with 24 hours supply temperature graph
5. Inside the heatpump icon is an (i) button that will show statistics on minutes in each mode and power consumption.
	
## Worklist
Using the [releases](https://github.com/supersjellie/openhab-nibe-heatpump/releases) in github now
Using the [issues](https://github.com/supersjellie/openhab-nibe-heatpump/issues) in github now
(I'm not using branches for work in progress (i.e. latest milestone), so download a release for a stable version)

## Preperation
1. Have an openhab installation :grin:
2. Own a NIBE heatpump :grin:
3. Have a NIBE uplinke account (or create it, it's free), see [documentation](https://www.nibeuplink.com/)
4. Install in openhab the NIBE uplink binding, see [documentation](https://www.openhab.org/addons/bindings/nibeuplink/)
5. Install in openHAB the javascript scriping engine, see [documentation](https://www.openhab.org/addons/automation/jsscripting/)
	* This installs the new ECMAscript 2021+ version with OH libraries that somewhere in the future will replace the old (now default) js-script. If you would like to stick to the default build-in version you'll have to change the rules a bit	

## Create thing
1. Create a new thing
2. Choose the NIBE binding
3. Choose your model heatpump
4. Add the credentials from your uplink account
5. Save the thing 
6. Don't close it

## Create equipment
1. If it's closed, navigate to your fan thing
2. Move to the tab 'channels'
3. Use button 'Add equipment to model
4. Select all channels and add-on
5. Openhab will create/add equipment and items
6. Remember the name of the group equipment, you need it for the widget configuration

## Widget installation
1. Copy all images in the project folder widget-images to your openhab configuration 'html' folder 
1. Add a new widget
2. Remove default code
3. Copy and past the yaml in the widget code of this project and save

## Refresh Rule installation
openHAB doesn't update loaded charts (even when in popups). The trick is to add a key to the f7-chart dependend on a periodically changing item. Since the popop shows a day chart every hour is more  than enough. You might already have encountered this challenge. So there's two options:
1. If you already have such an item. Look in the chart yaml for the key: = "heatpump" + items.timertick_hour.state and replace it with your item.
2. If this is the first time you need this
	* Create a new rule
	* Give it a name and label (I used timertick for both, but it doesn't matter what you choose)
	* Save it
	* Now change from the GUI config to the code tab
	* Replace code with the timertick_rule yaml in folder rules 
	* save it
	* run it (this will create the 'timertick_hour' item and initialize it.

## Heatpump Derived Data Rule installation
The create the derived data a rule is needed.
1. Create a new rule
2. Give it a name and label (I used heatpump for both, but it doesn't matter what you choose)
3. Save it
4. Now change from the GUI config to the code tab
5. Replace code with the heatpump_rule yaml in folder rules 
6. save it

## Widget usage/configuration
1. Add a new widget to your dashboard/page
2. Set the props and save
	* Easiest is selecting only the equipment (items will be found on default names)
	* If you choose other names for the items, you can also select the seperate items

## Usage
1. Add the widget 'widget_fan' to your dashboard or page. Remember it's intended for tiled dashboards (so small)
2. Set the props of the widget
	* Option 1 : Only set the equipment items (don't change seperate items)
	* Option 2 : Only set (all!) the seperate items (don't set equipment item)
3. You're done
	* It will show the status and info of your heatpump's operation
	* Press outside temperature, the (i) logo or any of the pumping speeds to get a 24 hours graph in a popup.
	* Icons will change according to settings, remember. The fan will respond immediatelly, the GUI with a 30 second delay (according to thing update frequency)

## Versions
* 1.0 Initial version
* 1.1 Added temperature popup
* 1,2 Added brine, boiler, heat/cool temperature popup
* 1.3 Calculate rolling sum of minutes active in each mode in 24 hrs
* 1.4 Calculate rolling sum of minutes active in each mode in last 3 hrs
* 1.5 Change to standard rule using helper methods and published on github
	
## Code
The code is pretty standard. Things you might find interesting for changes
1. I tried the cache functionality of the ECMA2021, big advantage is it survives a rule save/init. Don't put a JSON object on it, it produces errors when retrieved elements. Workaround for this issue, put it on as a string and convert it to the object.
2. To get running 24 hours totals it stores a value each hour. When a new hour is complete it overwrites the old value. The running hour is extrapolated.
3. The binding is not 100% consequent. So sometimes the unit must be ditched for calculation.
4. Embedded popovers in a widgets are a bit strange
	* Add a popoverOpen to the caller, create a unique name. Start with a "."!
	* Add a slot with a f7-popover in the calling item
	* Give it a class with the unique name, but without the first "." (and if present replace the other point by a space)!
	* Now they should connect/work. Always save before testing, openHAB will leave the widgetscreen when misconfigured.
5. OpenHAB charts don't update/refresh when loaded. The trick is to add a key dependend on a item that changes periodically. I've got two items for 15 minutes and a hour called timertick and timertick hour. If you put a key: = "fan" + items.timertick_hour.state (name must be unique) in the f7-card the chart will reload/refresh. This also applies to popups with charts (they're loaded hidden) 
6. Rules in ECMAscript allow 'let' but don't use it at main level (alway var). Reason is the script object is reused.
7. Using the historicState API generates fake/extrapolated RRD4J points that interfere with the running average. So the raw data is requested with actions.HTTP.sendHttpGetRequest on the persistance API.
8. The trick for optional properties in the widget.
	* Have a (standard) group item property with no default
	* Have a detail item property with a default value with the postfix like '_itemname'
	* Use in the widget formulas like 'items[props.group+props.detail].state'
	* If you select a group item 'yyy' it will add up to 'yyy_itemname'. So with only setting the group all the details items will be set
	* if you leave the group empty and select a item like 'qqq_itemname' it will add up to 'qqq_itemname'. So just as you selected ignoring the group.