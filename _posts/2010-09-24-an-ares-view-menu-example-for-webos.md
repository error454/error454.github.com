---
title: An Ares View Menu Example for WebOS
author: error454
excerpt: How to create a viewMenu widget.
layout: post
permalink: /2010/09/24/an-ares-view-menu-example-for-webos/
categories:
  - Ares
  - WebOS
tags:
  - ares
  - example
  - tutorial
  - view menu
  - viewmenu
  - webos
  - widget
---
<a href=''><img src='{{ site.url }}/assets/uploads/2010/09/menu.png' alt=''></a>

Want a menu that looks like this?

Are you using Ares as your development environment?

Are you frustrated because this menu doesn't exist in the list of Widgets?

Read on for a full example of how to easily implement this widget along with how to tie in the functionality for the buttons.

## The viewMenu Widget

<a href="http://developer.palm.com/index.php?option=com_content&view=article&id=1994&Itemid=328#.viewMenu" target="_blank">This</a> is a special widget that is not created in the same way that other widgets are.  There isn't a viewMenu UI Widget in the Palette to drag onto your scene.  The link above explains more details about this, what you need to know is that this menu is declared programatically.

We'll dive right in by creating a new Ares project and modifying main-assistant.js by adding the createMenu function:

<pre>function MainAssistant(argFromPusher) {
}

MainAssistant.prototype = {
	setup: function() {
		Ares.setupSceneAssistant(this);
		this.createMenu();
	},
	cleanup: function() {
		Ares.cleanupSceneAssistant(this);
	},
	createMenu: function() {
		this.controller.setupWidget(Mojo.Menu.viewMenu,
			this.attributes = {
				spacerHeight: 50,
			},
		this.model = {
			visible: true,
			items: [
				{
					items: [
						{ label: Mount Image, command: mount, width: 160 },
						{ label: Create Image, command: create, width: 160 }
					],
					toggleCmd: mount
				}
			]
		});
	}
};
</pre>

## The createMenu function

The createMenu function is quite simple, all it does is call <a href="http://developer.palm.com/index.php?option=com_content&view=article&id=1874&Itemid=244#.setupWidget" target="_blank">Mojo.Controller.SceneController.</a>**<a href="http://developer.palm.com/index.php?option=com_content&view=article&id=1874&Itemid=244#.setupWidget" target="_blank">setupWidget</a>**<a href="http://developer.palm.com/index.php?option=com_content&view=article&id=1874&Itemid=244#.setupWidget" target="_blank">(name, attributes, model)</a>.  Let's walk through each parameter.

### name

<p style="padding-left:30px;">
  The name of the widget we are configuring, in this case it is Mojo.Menu.viewMenu
</p>

### attributes

<p style="text-align:left;padding-left:30px;">
  A list of the attributes for this widget.  There are only 2 choices for the <a href="http://developer.palm.com/index.php?option=com_content&view=article&id=1994&Itemid=328#.viewMenu" target="_blank">viewMenu </a>widget.  As you can see on line 15, I have specified spacerHeight to be 50.  What this does is push down the contents of your scene by 50 pixels.  The viewMenu widget is overlayed on the scene, so with the default spacerHeight of 0, the content of the scene will be obscured by the viewMenu.
</p>

### model

<p style="padding-left:30px;">
  The model defines the items in the menu.  You'll notice that above, I am creating an array within another array.  The reason for this is to create what is called a <em>toggle group</em>.  If I had put the menu items in only 1 array, the menu would like like this.
</p>

<a href=''><img src='{{ site.url }}/assets/uploads/2010/09/non-group.png' alt=''></a>

<p style="padding-left:30px;">
  The toggle group makes the menu behave like the radio button widget.  Notice that for each item, we have defined:
</p>

<p style="padding-left:30px;">
  <strong>label  </strong>What you see.
</p>

<p style="padding-left:30px;">
  <strong>command  </strong>A reference for when we make the button come to life.
</p>

<p style="padding-left:30px;">
  <strong>width  </strong>Calculated so that the sum of all buttons is the width of the screen (320).
</p>

<p style="padding-left:30px;">
  Finally, line 25 sets the default state of the toggle group so that mount is selected.
</p>

## Making the Buttons Work

In this example, I am going to make the buttons display/hide a panel on the screen.  Based on my scene layout, you should be able to see quite easily what I am doing.

<img class="alignnone size-full wp-image-590" title="scene" src="{{ site.url }}/assets/uploads/2010/09/scene.png" alt="" width="169" height="153" />

When the mount button is pressed, I will hide **panelCreate** and show **panelMount** and vice versa.  The code to do so uses the command references from our items and looks like this.

<pre>handleCommand: function(event) {
		if (event.type == Mojo.Event.command) {
			switch (event.command) {
				case mount:
					this.$.panelCreate.setShowing(false);
					this.$.panelMount.setShowing(true);
					break;
				case create:
					this.$.panelMount.setShowing(false);
					this.$.panelCreate.setShowing(true);
					break;
				default:
					break;
			}
		}
	}
</pre>

## Full Source

The full source is below.

<pre>function MainAssistant(argFromPusher) {}

MainAssistant.prototype = {
	setup: function() {
		Ares.setupSceneAssistant(this);
		this.createMenu();
	},
	cleanup: function() {
		Ares.cleanupSceneAssistant(this);
	},
	createMenu: function() {
		this.controller.setupWidget(Mojo.Menu.viewMenu, this.attributes = {
			spacerHeight: 50,
		},
		this.model = {
			visible: true,
			items: [
				{
					items: [
						{ label: Mount Image, command: mount, width: 160 },
						{ label: Create Image, command: create, width: 160 }
					],
					toggleCmd: mount
				}
			]
		});
	},
	handleCommand: function(event) {
		if (event.type == Mojo.Event.command) {
			switch (event.command) {
				case mount:
					this.$.panelCreate.setShowing(false);
					this.$.panelMount.setShowing(true);
					break;
				case create:
					this.$.panelMount.setShowing(false);
					this.$.panelCreate.setShowing(true);
					break;
				default:
					break;
			}
		}
	}
};
</pre>