---
layout: default
title: Create a Hosted Web App
permalink: /en-US/win10/MigrateToUWP.htm
lang: en-US
---

#Migrating your Windows 8.1 App to a Universal Windows Platform (UWP) app for Windows 10

Migrating an HTML/JavaScript app from Windows 8.1 to the Windows 10 can be accompanied by a few challenges. Learn what common roadblocks app developers are most likely to encounter when migrating their Windows 8.1 apps, and how to address them.

##Security Model Changes - The new Windows 10 security model for HTML/Javascript apps.

In Windows 8 and 8.1, HTML/JavaScript apps using the default **ms-appx:///** protocol applied a Microsoft specific “SafeHTML” security model, which imposed restrictions around changing innerHTML or dynamically adding certain types of attributes. Performing these operations in our Windows 8.1 App would have required the use of additional SafeHTML APIs 

'''js

// SafeHTML APIs in use 
var nextDiv = document.createElement(“DIV”); 
nextDiv.innerHTML = toStaticHTML(myDiv.innerHTML); 
MSApp.execUnsafeLocalFunction(function() { var nextDiv = document.createElement(“DIV”) nextDiv.innerHTML = myDiv.innerHTML; }); 
'''

In Windows 10 the SafeHTML security model has been removed. When Migrating an app we should remove all references to the SafeHTML APIs, or if too numerous, feature detect them and provide a shim for environments where they don't exist. 

'''js
// HTML injection restrictions from Windows 8 have been removed in UWP. 
// Shim these functions since they no longer exist in UWP
if (!window.toStaticHTML) { window.toStaticHTML = function (text) { return text; }; }
if (window.MSApp && !window.MSApp.execUnsafeLocalFunction) { MSApp.execUnsafeLocalFunction = function (c) { c(); }; } 
'''

The new security model for Windows 10 apps using the default **ms-appx:///** protocol, applies an "unsafe inline" CSP directive by default. CSP is a standardized [W3C security Model]{http://w3c.github.io/webappsec-csp/}.

Under the new Security Model, any inline script tags defined within an App’s HTML files will not execute. 
'''html
<!-- This will not execute --> 
<script> var myDiv = document.body.querySelector(“#mydiv”); </script> 
'''

Instead, we should move the script from our inline script tags into a .js file that our same HTML file can load. 
'''html
<script src=”./js/default.js”></script> 
'''

For more information and links to documentation on the new security model, visit: https://msdn.microsoft.com/en-us/library/windows/apps/dn705792.aspx?f=255&MSPPError=-2147217396#new_security_model 

##Settings Changes - Invoking Settings for Win10 HTML/Javascript apps.

In Windows 8 and 8.1, developers were encouraged to make their app settings accessible through the *Settings* charm in the Windows Charms bar. 

In Windows 10, there is no charms bar. It is reccomended that UWP apps add their own app UI for users to invoke their settings view. If your app is using WinJS, you may want to include a command for settings in your WinJS AppBar.

##Migrating to WinJS 4.X

When migrated an App that was using WinJS 2.0 or WinJS 2.1, you will probably notice that the look and feel of the 2.X version of WinJS does not match the new Windowws 10 design language enjoyed by the first party UWP apps in the Windows store.

To upgrade the look and feel of you WinJS App to follow the Windows 10 design language, you can grab the latest version of WinJS 4.X. WinJS is a now an open source project and the latest version can be obtained [from here](http://try.buildwinjs.com/download/GetWinJS/)

Some WinJS design concepts and controls have been changed more significantly than others. This can lead to non-trivial work when migrating some apps. See the WinJS [change log](https://github.com/winjs/winjs/wiki/Changelog) for the high level details on whats changed. Below are some tips for adjusting to two of the biggest changes in WinJS 4.X

###Changes to WinJS styling of instrinsic controls###

The `ui-light` and `ui-dark` stylesheets in WinJS 2 contained CSS rules for both the set of controls in the WinJS UI Toolkit, and many browser controls like `<button>`,  `<input type="range">` and `<h1>` elements. 

Outside of some generic styles applied to  <body>  and  <html> , WinJS 4.X does not globally style any browser controls such as  <button> , or apply any font styles. Instead, you must opt-in to styling for each element you create, similar to Twitter Bootstrap.

For example, take button elements:
- In WinJS 2.X, all `<button>` or `<input>` button elements would have the styles for the Modern Windows look and feel automatically applied. 
- In WinJS 4.X, a should use the following button classes on any `<button>` or button `<input>` elements that they want to give the Modern Windows look and feel.
  	- `win-button` - Base button class
  	- `win-button-primary` - Add this class to a button in addition to `win-button` to make it stand out
  	- `win-button-file` - Add this class to a button in addition to `win-button` when its type is `file`
      
	  ```html
	  <button class='win-button'>Default button</button>
	  <input type='button' class='win-button win-button-primary' value='Primary button' />
	  <input type='file' class='win-button win-button-file' />
	  ```
- A detailed list for all the new opt-in styles can be found [here](https://github.com/winjs/winjs/wiki/Styling-HTML-Controls)

###Changes to the WinJS AppBar###

At a glance the WinJS 4.0 AppBar may look similar to previous AppBar implementations, but some aspects of the design have changed significantly since WinJS 2.0. To facilitate these design changes, key AppBar API's have been renamed, replaced, or removed. The extent of how many changes to the AppBar that an app may require will depend on which features off the AppBar was using, but it is likely that most appbars will require at least a few changes.

Here are the important details on how to Migrate your app from using the WinJS 2.0 AppBar, to the WinJS 4.X AppBar.

**Paradigm Shift**
	• In WinJS 2.0, the AppBar control doesn't provide any dedicated UI for opening or closing it. Additionally, when closed, the AppBar was hidden completely from view and out of a user's reach. 
		
	• In Windows 8.1 apps, the paradigm for end users to invoke the AppBar was either (1) swiping a finger across the top or bottom of the screen, or (2) right clicking an empty region of the App. When either action occurred, Apps for 8.1 would trigger a custom event causing the  WinJS 2.0 AppBar to open/close itself.
	
	• In Apps for Windows 10, the paradigm has changed. Right clicking or edge swiping will no longer fire an event to open the AppBar. Instead The WinJS 4.X AppBar provides its own UI for users to open and close it.
	
	• Additionally, the AppBar can be configured, via the new `AppBar.closedDisplayMode` property to appear in 4 distinct visual states while closed.
		- full
		- compact (default)
		- minimal
		- none (same as WinJS 2.0 AppBar)
	
	For more information on the design of the new AppBar, please visit:
	https://msdn.microsoft.com/en-us/library/windows/apps/hh465296.aspx
	
**Renamed API's:**
	• Methods
		- The hide() method was renamed to close()
		- The show() method was renamed to open()
	• Events
		- Renamed the onafterhide event property to onafterclose
		- Renamed the onaftershow event property to onafteropen
		- Renamed the onbeforehide event property to onbeforeclose
		- Renamed the onbeforeshow event property to onbeforeopen
	• CSS classes
		- Renamed the  .win-appbar-hidden CSS class to .win-appbar-closed.

	If your Windows 8.1 app is using any of these API's by their previous name, you simply need to rename them to the new value and they will continue to work the same as before.

**Removed API's:**
	• Properties
		- The `hidden` property has been removed. If apps that were using the hidden property, should use the new `opened` property instead.  Unlike the hidden property, the opened property is both get and set. 
			'''js
			// if(appBar.hidden) { return true }
			if(!appBar.opened){ return true }
			'''
		- The `commands` property was removed. If your app was using the commands property to set AppBar commands programatically, you should use the new `data` property instead. 
			**difference**
			- The commands property was set only, but the data property is get/set. 
			- Unlike the commands property which took an Array of Instantiated AppBarCommands, the AppBar.data property only accepts a WinJS.Binding.List of Instantiated AppBarCommand objects. 
			'''js
			var cmds = [
					new WinJS.UI.AppBarCommand(null, {icon: 'add', label'add'}),
				    	new WinJS.UI.AppBarCommand(null, {icon: 'delete', label'delete'})
				   ];
			// appBar.commands = cmds;
			appBar.data = new WinJS.Binding.List(cmds);
			'''
			
		- The `disabled` property has been removed. Apps that were using the disabled property can set the AppBar's `closedDisplayMode` property to "none", and close the AppBar to hide it completely, removing any ability for an end user to interact with it.
		- The `sticky` property has been removed. Apps that were using the `sticky` property should instead set the new `closedDisplayMode` property to "compact" or "full".
		- The `layout` property has been removed. Apps that were setting the `layout` property to "custom", should host that content inside of AppBarCommand(s) whose `type` property is set to "content".
		- [Example](https://msdn.microsoft.com/en-us/library/windows/apps/hh780658.aspx) of AppBar with "content" commands for hosting custom conent.

	• Methods	
		- showCommands() and hideCommands() have been removed from the AppBar. Apps that were using showCommands() or hideCommands() should instead use showOnlyCommands() to describe the set of commands in the AppBar that should be visible. All commands not included in the array parameter will be implicitly hidden.
		'''js
		var cmdAdd = new WinJS.UI.AppBarCommand(null, {icon: 'add', label'add'});
		var cmdDel = new WinJS.UI.AppBarCommand(null, {icon: 'delete', label'delete'});
		appBar.data = new WinJs.Binding.List([cmdAdd, cmdDel]);
		
		// Hide Commands
		// appBar.hideCommands([cmdAdd, cmdDel]);
		appBar.showOnlyCommands([]);
		
		// Show Commands
		// appBar.showCommands
		appBar.showOnlyCommands([cmdAdd, cmdDel]);
		'''
		
	• CSS classes
	      	- Removed the `.win-commandlayout` CSS class. This class name was obsolete with the removal of the "layout" property.

**New layout and rendering for AppBarCommands**

	- In WinJS 2.X apps could control whether commands were placed on the left side or right side of the AppBar via the AppBarCommand's `section` property via values of either "global" or "selection". 
	- In WinJS 4.X the strategy of laying out commands to either the left or the right is no longer suppported, and the values of "global" and "selection" for the AppBarCommand `section` propety have been deprecated. 
		- Instead commands are laid out in DOM order, into either: 
			- a single row of the action area (always left aligned except in RTL mode), 
			- a single column of the overflow area.
		- Developers can designate which area a command should go to by setting the AppBarCommand `section` property to 
			- primary (action area) (this is the default value)
			- secondary (overflow area)
		- Some primary commands in the actionarea can overflow into the overflow area if there is not enough room to fit all primary commands. 
		- Developers can set the AppBarCommand `priority` property to designate the order in which commands should overflow.
			- The `priority` property will accept any non negative number. 
			- Commands with a weaker priority will overflow first. Priority 0 is considered to be stronger than Priority 1, andd Priority 1 is stronger than Priority 2.
			- Commands assigneda the same `priority` value will be grouped together and overflow at the samee time.