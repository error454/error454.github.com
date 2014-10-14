---
title: The 10-minute guide to Ares for WebOS
author: error454
layout: post
permalink: /2010/08/06/the-10-minute-guide-to-ares-for-webos/
categories:
  - WebOS
tags:
  - ares
  - webos
---
This is the 10-minute reference that I was looking for when I started using Ares for my WebOS projects.  Topics covered:

*   UI Basics
*   Widget Fundamentals
*   Using Widgets the Ares Way



## UI Basics

<p style="padding-left:30px;">
  After starting a new project, you will have a fresh canvas full of potential!  Your screen will look like this.
</p>

<p style="padding-left:30px;">
  <a href="{{ site.url }}/assets/uploads/2010/08/ui-basics-1.png"><img class="size-medium wp-image-256" title="An empty canvas" src="{{ site.url }}/assets/uploads/2010/08/ui-basics-1.png?w=300" alt="" width="300" height="259" /></a>
</p>

<p style="padding-left:30px;">
  If you haven't familiarized yourself with the UI then let's take a quick tour together.
</p>

<h3 style="padding-left:30px;">
  Left Panel
</h3>

<p style="padding-left:60px;">
  On the left-hand side, you have a menu that toggles the left pane between 3 views.  The 3 views are:
</p>

<p style="padding-left:60px;">
  <img class="alignnone size-full wp-image-257" title="ui basics 2" src="{{ site.url }}/assets/uploads/2010/08/ui-basics-2.png" alt="" width="298" height="34" />
</p>

<ul style="padding-left:60px;">
  <li>
    <strong>Palette  </strong>This view contains all of the pre-built widgets that can be used in your app.  Buttons, check boxes and text boxes are just some of the choices here.  You will notice that this panel is organized by widget type, with visible widgets listed first and then things like system services coming later.  Everything under the Palette panel can be added to your application by dragging the widget onto the scene.
  </li>
  <li>
    <strong>View  </strong>Don't dismiss this panel, it is incredibly useful!  The view panel shows you the hierarchical layout of your widgets.  If you are wondering why your widgets don't quite look right, this panel can be invaluable.  For instance, you can quickly see whether a button is contained inside of a horizontal scroll panel. <p>
      <a href=''><img src='{{ site.url }}/assets/uploads/2010/08/ui-basics-6.png' alt=''></a></li> 
      
      <li>
        <strong>Files  </strong>This panel allows you to browse your projects and files, double clicking a file will open it in the viewer.
      </li></ul> 
      <h3 style="padding-left:30px;">
        Right Panel
      </h3>
      
      <p style="padding-left:60px;">
        On the right-hand side, you have a menu that toggles the right pane between 4 views. The 4 views are:
      </p>
      
      <p style="padding-left:60px;">
        <img class="alignnone size-full wp-image-258" title="ui basics 4" src="{{ site.url }}/assets/uploads/2010/08/ui-basics-4.png" alt="" width="262" height="31" />
      </p>
      
      <ul style="padding-left:60px;">
        <li>
          <strong>Settings  </strong>This panel displays the settings for the selected widget.  Each widget has a unique set of properties.
        </li>
        <li>
          <strong>Styles  </strong>This panel displays the style for the selected widget, allowing you to change padding, background color and opacity on a per-widget basis.
        </li>
        <li>
          <strong>Events  </strong>This panel displays a list of events that can be hooked for the given widget.  Events are used as a way to perform an action based on user interaction with a widget.  For instance, a Button has the following events. <a href=''><img src='{{ site.url }}/assets/uploads/2010/08/ui-basics-5.png' alt=''></a>
          
          <p>
            Clicking on the icon that looks like a piece of paper will automatically add a new event handler for a given event and will create an empty event handler method in your code!</li> <li>
              <strong>Help  </strong>This panel display help for a selected widget.  As of this time, only non-visible widgets contain help entries.
            </li></ul> 
            <h3 style="padding-left:30px;">
              Bottom Panel
            </h3>
            
            <p style="padding-left:60px;">
              At the bottom of the screen you'll see the Non-Visual Components section.  When you drag services and other non-visual widgets to the canvas, this is where they show up.
            </p>
            
            <h3 style="padding-left:30px;">
              UI Navigation Tips
            </h3>
            
            <ul>
              <li>
                <ul>
                  <li>
                    When a widget is selected, the ESC key will select the parent widget
                  </li>
                  <li>
                    The View tab makes selecting the correct widget infinitely easier
                  </li>
                  <li>
                    When dragging a widget across the UI, the dragging pop-up shows you which container you are currently hovering over. This makes it much easier to place widgets
                  </li>
                  <li>
                    Dragging/dropping in the View menu does nothing
                  </li>
                </ul>
              </li>
            </ul>
            
            <h2>
              Widget Fundamentals
            </h2>
            
            <h3 style="padding-left:30px;">
              Scrollers
            </h3>
            
            <p style="padding-left:60px;">
              One thing you will almost always want to do is add a Scroller widget (Palette -> Layout -> Scroller) as the first item in your scene.  <img class="size-full wp-image-263 alignleft" title="scroller" src="{{ site.url }}/assets/uploads/2010/08/scroller.png" alt="" width="99" height="37" /> If you do use a scroller widget then you can simply use your mouse-wheel to scroll the scene up/down while designing.  If you don't use a scroller widget and your scene extends beyond the height of the screen, you will have to hide widgets to be able to reach the widgets further down on the screen.
            </p>
            
            <p style="padding-left:60px;">
              Once you add the Scroller, you may notice that it does not fill the scene.  To easily make the Scroller fill the scene, you can use the right-hand panel (Settings -> Sizing Tools -> Maximize).
            </p>
            
            <h3 style="padding-left:30px;">
              Widget Names
            </h3>
            
            <p style="padding-left:60px;">
              Even if you never plan on sharing your code, it is good practice to give useful names to the widgets that you will be referencing often (Buttons, labels, services).  It is important that you do this before you start adding event handlers to your widgets, otherwise you may have to go and change the names of any event handlers attached to the widget.  Many people use the objectVerbNoun naming scheme for things that perform an action (buttonCloseAlert or buttonSendData) and the objectNoun scheme for things like labels (labelAddress).
            </p>
            
            <p style="padding-left:60px;">
              <a href="{{ site.url }}/assets/uploads/2010/08/naming.png"><img class="alignnone size-full wp-image-264" title="naming" src="{{ site.url }}/assets/uploads/2010/08/naming.png" alt="" width="263" height="191" /></a>
            </p>
            
            <p style="padding-left:60px;">
              <a href="{{ site.url }}/assets/uploads/2010/08/naming.png"></a>Note: I usually don't rename my scrollers and headers since I never refer to them in the actual code.
            </p>
            
            <h2>
              Using Widgets the Ares Way
            </h2>
            
            <p style="padding-left:30px;">
              Ares makes widgets easy to use.  To illustrate how to interact with widgets in Ares, I have created a simple project.  You should have no trouble creating the same project by looking at this screenshot.
            </p>
            
            <p style="padding-left:60px;">
              <a href="{{ site.url }}/assets/uploads/2010/08/sample-project.png"><img class="alignnone size-medium wp-image-265" title="sample project" src="{{ site.url }}/assets/uploads/2010/08/sample-project.png?w=300" alt="" width="300" height="259" /></a>
            </p>
            
            <p style="padding-left:30px;">
              Here we have a very simple scene, a scroller, header, label and button.  The top right panel under the Common section is where you make your basic modifications like changing the name and label of a widget.  For instance, on my header, I changed the <strong>label </strong>property to Widget Fundies.  I also changed the <strong>label </strong>property of labelForce so that it is empty and buttonUpdateForce to The Force.
            </p>
            
            <h3 style="padding-left:30px;">
              Updating Widgets Programatically
            </h3>
            
            <p style="padding-left:60px;">
              Select buttonUpdateForce and add an event handler for ontap:
            </p>
            
            <p style="padding-left:60px;">
              <img class="size-full wp-image-267 alignnone" title="handler" src="{{ site.url }}/assets/uploads/2010/08/handler.png" alt="" width="268" height="173" />
            </p>
            
            <p style="padding-left:60px;">
              After clicking, you will be whisked away to the code view where you will find this event handler:
            </p>
            
<pre>
buttonUpdateForceTap: function(inSender, event) {

}
</pre>

Let's make our button do something by modifying line 2 as shown below.


<pre>
buttonUpdateForceTap: function(inSender, event) {
	this.$.labelForce.setLabel(Is Strong!);
}
</pre>


Because the <strong>buttonUpdateForceTap </strong>event handler is defined in the MainAssistant.prototype, <strong>this </strong>refers to MainAssistant.  Adding the dollar sign <strong>this.$</strong> refers to the collection of objects inside MainAssistant, meaning that every widget contained in main-chrome.js is available through this.$.  Although you cannot see the code generated, when you switch to the GUI view of main-assistant.js, you are actually modifying main-chrome.js.  You can modify any common property using this method, here are a couple examples.


<pre>
//Get the current value of the label
var currentValue = this.$.labelForce.getLabel();

//Change the label of our button
this.$.buttonUpdateForce.setLabel(Join the Dark Side);

//Change the class of our button
this.$.buttonUpdateForce.setButtonClass(secondary);

//Make our button disappear
this.$.buttonUpdateForce.setShowing(false);

//Get the value of showing
var isShowing = this.$.buttonUpdateForce.getShowing();
</pre>

That is all for this brief introduction, go forth and program!
