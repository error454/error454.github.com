---
title: 'ShiVa Native Projects for Android  A Guide for the Forgetful'
author: error454
layout: post
permalink: /2012/04/14/shiva-native-projects-for-android-a-guide-for-the-forgetful/
tagazine-media:
  - 'a:7:{s:7:"primary";s:66:"https://mobilecoder.files.wordpress.com/2012/04/export-project.jpg";s:6:"images";a:4:{s:62:"https://mobilecoder.files.wordpress.com/2012/04/stk-export.jpg";a:6:{s:8:"file_url";s:62:"https://mobilecoder.files.wordpress.com/2012/04/stk-export.jpg";s:5:"width";s:3:"646";s:6:"height";s:3:"467";s:4:"type";s:5:"image";s:4:"area";s:6:"301682";s:9:"file_path";s:0:"";}s:66:"https://mobilecoder.files.wordpress.com/2012/04/export-project.jpg";a:6:{s:8:"file_url";s:66:"https://mobilecoder.files.wordpress.com/2012/04/export-project.jpg";s:5:"width";s:4:"1029";s:6:"height";s:3:"787";s:4:"type";s:5:"image";s:4:"area";s:6:"809823";s:9:"file_path";s:0:"";}s:58:"https://mobilecoder.files.wordpress.com/2012/04/assets.jpg";a:6:{s:8:"file_url";s:58:"https://mobilecoder.files.wordpress.com/2012/04/assets.jpg";s:5:"width";s:3:"212";s:6:"height";s:2:"75";s:4:"type";s:5:"image";s:4:"area";s:5:"15900";s:9:"file_path";s:0:"";}s:58:"https://mobilecoder.files.wordpress.com/2012/04/userai.jpg";a:6:{s:8:"file_url";s:58:"https://mobilecoder.files.wordpress.com/2012/04/userai.jpg";s:5:"width";s:3:"299";s:6:"height";s:3:"263";s:4:"type";s:5:"image";s:4:"area";s:5:"78637";s:9:"file_path";s:0:"";}}s:6:"videos";a:0:{}s:11:"image_count";s:1:"4";s:6:"author";s:8:"11758919";s:7:"blog_id";s:8:"11929434";s:9:"mod_stamp";s:19:"2012-04-14 21:33:13";}'
publicize_results:
  - 'a:1:{s:7:"twitter";a:1:{i:37590404;a:2:{s:7:"user_id";s:8:"error454";s:7:"post_id";s:18:"191376317382524928";}}}'
categories:
  - Android
  - Java
  - Shiva 3D
tags:
  - android
  - C++
  - java
  - native
  - shiva
---
This article covers the basics of exported Android ShiVa projects.  If you are trying to integrate Java or C libraries, for instance the ScoreLoop API, the following information could come in handy.  I had to stumble through this process with the scattered bits of documentation and I get tired of re-learning it every time I start a new project.  The article assumes basic familiarity with Eclipse, Java, C, Android and JNI.

Most of this content is based on the file that Stonetrip provides, on windows it is:
<!--more-->
<pre>
C:\Program Files (x86)\Stonetrip\ShiVa Authoring Tool\Data\Windows\Windows\Build\S3D SDK - Readme.txt
</pre>



## STKs & Project Authoring

It's important to understand how things fit together in ShiVa so that you don't waste your efforts.  The ShiVa project eco-system works like this.  From the ShiVa Editor, you export an .stk file, this is your actual game.

<a href=''><img src='{{ site.url }}/assets/uploads/2012/04/stk-export.jpg' alt='Exporting an stk file, this is your game.'></a>

Once your .stk is exported, you open it up in the Authoring Tool and you export as a Project instead of an APK package.

<a href='{{ site.url }}/assets/uploads/2012/04/export-project.jpg'><img src='{{ site.url }}/assets/uploads/2012/04/export-project.jpg?w=1024' alt='Export as Project, be sure to set things in the Build tab first!'></a>

Before you export as a project, you need to choose your Build settings correctly.  For instance if you're going to use Network then be sure to check that box, if you're using openGL 1.1 be sure to check that etc.  It's possible to go back and change these later but it can be a bit tedious if you're not familiar with where the parameters materialize in the exported project.

When you export as a project, you're given a zip file.  The zip file contains an eclipse project, once you import this project into eclipse you are done exporting your project from the authoring tool for good.

## Eclipse Project Details

Take a look in the assets folder once you've imported the project.  Take special note that the S3DMain.smf file is actually just an .stk file that is renamed.

<a href='{{ site.url }}/assets/uploads/2012/04/export-project.jpg'><img src='{{ site.url }}/assets/uploads/2012/04/assets.jpg' alt='The assets folder, S3DMain.smf is just an .stk file renamed'></a>

Every time you update your game, you are going to export the .stk, rename it to S3DMain.smf and overwrite this file in the assets folder.

To build, you can't just click the play button, you have to build with Ant.

1.  Window->Show View->Ant
2.  Drag the build.xml into the Ant window
3.  Double click on Build Debug APK

## Interfacing ShiVa with Other Code

There are 2 basic scenarios that I'll cover here.

### Scenario 1: Call C/C++/Java Code from ShiVa

The way you accomplish this is by setting a listener for a ShiVa user event.  In this example, I have the following userAI:

<a href='{{ site.url }}/assets/uploads/2012/04/export-project.jpg'><img src='{{ site.url }}/assets/uploads/2012/04/userai.jpg' alt='A userAI with 2 handlers'></a>

What we're going to do is make it so anytime these handlers are called in your game, some Java code will run as a result.  As far as the order of operations, my logging has told me that once the handler is called, the native code is ran and then the remainder of the LUA handler runs.  Here is how to setup a listener for a handler.

**Step 1.** Open up S3DClient.cpp and search for **BEGIN\_JNI\_INSTALL\_EVENT\_HOOKS, **you seriously can't miss it.

**Step 2.** Add listeners for each handler.  Here's what mine looks like:

<pre>//----------------------------------------------------------------------
// @@BEGIN_JNI_INSTALL_EVENT_HOOKS@@
//----------------------------------------------------------------------
S3DClient_InstallCurrentUserEventHook( "achievementAI", "onIncrementAchievement", incrementAchievement, NULL );
S3DClient_InstallCurrentUserEventHook( "achievementAI", "onAwardAchievement", awardAchievement, NULL );
//----------------------------------------------------------------------
// @@END_JNI_INSTALL_EVENT_HOOKS@@
//----------------------------------------------------------------------
</pre>

Note the parameters (userAI, handler, function, NULL).  The functions named in the above will be called whenever the handlers are called.

**Step 3.** Write the JNI function (also in S3DClient.cpp just above the definition for engineInitialize).  If you've never done this before then buckle up and read some <a href="http://java.sun.com/docs/books/jni/html/jniTOC.html" target="_blank">sun articles</a>.  Here is the JNI for incrementAchievement, the goal is to call a Java function in my main class called **incrementAchievement()**.

<pre>void incrementAchievement(unsigned char _iArgumentCount, const void *_pArguments, void *_pUserData){
	JNIEnv *pJNIEnv = GetJNIEnv();
	if (pJNIEnv){
		if ( _pArguments && ( _iArgumentCount  0 ) ){
			const S3DX::AIVariable *pVariables = (const S3DX::AIVariable *)_pArguments ;

			for ( uint8_t i = 0 ; i  _iArgumentCount ; i++ ){
				//We only want strings
				if(pVariables[i].GetType() == S3DX::AIVariable::eTypeString){
					LOGI( "incrementAchievement returned string: %s", pVariables[i].GetStringValue() );

					//Find our main class so we can call the incrementAchievement function.  For me, my package name is:
					//com.hypercanestudios.acceleroketer
					//within that package I have a class called accelerocketer
					//so package name/class name and replace "." with "/"
					jclass pJNIActivityClass = pJNIEnv-FindClass ( "com/hypercanestudios/acceleroketer/acceleroketer" );

					if(pJNIActivityClass == NULL)
						LOGI("jclass was null!?!");
					else{
						//Now we have to find the function we're trying to call. We use the class defined above since the function
						//is a member of that class.  (Ljava/lang/String;) is the set of arguments that the function takes, a single String
						//V means that the function returns void
						//void incrementAchievement(String blah)
						//See table 3-2 http://docs.oracle.com/javase/1.3/docs/guide/jni/spec/types.doc.html#597
						jmethodID pJNIMethodID = pJNIEnv-GetStaticMethodID(pJNIActivityClass, "incrementAchievement", "(Ljava/lang/String;)V" );

						if(pJNIMethodID == NULL)
							LOGI("jmethodID was null!?!?");
						else{
							//Create a new string
							jstring arg;
							arg = pJNIEnv-NewStringUTF(pVariables[i].GetStringValue());

							//Call the method and pass the string parameter along
							pJNIEnv-CallStaticVoidMethod(pJNIActivityClass, pJNIMethodID, arg);

							//Free the string
							pJNIEnv-DeleteLocalRef(arg);
						}
					}
				}
			}
		}
	}
}</pre>

You could eliminate some of the checks against NULL in your final version but these are pretty helpful during the initial creation phase. When you crash due to a JNI error, you don't get a nice clean stacktrace so any debug you can print goes a long ways. Search stackoverflow.com for ways of debugging JNI stacks, you can use addr2line and get the function and line number.

**Step 4.** Write the Java function

<pre>public static void incrementAchievement(String id){
    Log.i("acceleroketer", "Hello World!!1 " + id);
}
</pre>

### Scenario 2: Call ShiVa code from Java

The other scenario is when you want to call down to your ShiVa code from a Java function.  Let's call achievementAI's onShowAchievement handler.

**Step 1.** Copy S3DX header files into your JNI folder, i.e. drag them into the JNI folder in eclipse and select copy.  On windows the headers are located in C:\Program Files (x86)\Stonetrip\ShiVa Authoring Tool\Data\Windows\Android\Build\S3DX

**Step 2.** Add the S3DXAIVariable header to your S3DClient.cpp:

<pre>//----------------------------------------------------------------------
// @@BEGIN_JNI_INCLUDES@@
//----------------------------------------------------------------------
#include
#include android/log.h
//----------------------------------------------------------------------
#include
#include
#include GLES/gl.h
//----------------------------------------------------------------------
#include "S3DXAIVariable.h"
#include "S3DClient_Wrapper.h"
</pre>

**Step 3.** In your Java code, define a native function, this will be used to call ShiVa.

<pre>public native static void showAchievementInShiva(String id);
</pre>

**Step 4.** In S3DClient.cpp implement the native method being sure to handle any parameters.

<pre>JNIEXPORT void JNICALL Java_com_hypercanestudios_acceleroketer_acceleroketer_showAchievementInShiva ( JNIEnv *_pEnv, jobject obj, jstring id ){
	//Convert java string 'id' to a const char * so we can pass it to shiva
	const char *nativeString = _pEnv-GetStringUTFChars(id, NULL);

	S3DX::AIVariable args[1];
	args[0].SetStringValue( nativeString );
	S3DClient_SendEventToCurrentUser( "achievementAI", "onShowAchievement", 1, (const void*)args);

	//Release string
	_pEnv-ReleaseStringUTFChars(id, nativeString);
}
</pre>

**Step 5. **Create the handler in your ShiVa userAI and print something useful to make sure it's working.

<pre>--------------------------------------------------------------------------------
function achievementAI.onShowAchievement ( id )
--------------------------------------------------------------------------------

	log.message ( "Got id: " .. id )

--------------------------------------------------------------------------------
end
--------------------------------------------------------------------------------
</pre>