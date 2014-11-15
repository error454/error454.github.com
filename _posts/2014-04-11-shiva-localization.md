---
title: ShiVa Localization
author: error454
layout: post
permalink: /2014/04/11/shiva-localization/
categories:
  - Interesting Problems At Work
  - 'Marketplace  Publishing'
  - Shiva 3D
tags:
  - csv
  - i18n
  - localization
  - lua
  - parser
  - shiva
  - spreadsheet
  - translation
  - xml
---
As we are nearing our first Steam release, we wanted to make a strong effort to localize our game in as many different languages as possible. This brought up an interesting problem involving workflow and tools that we've now solved and wanted to share. We wanted a solution that would allow:
<!--more-->
*   Multiple translators to work on a shared document at the same time
*   Automated conversion from the spreadsheet to the XML files used by the game engine
*   Easy usage inside the game engine
*   Forward compatibility with ShiVa 2.0 and its ability to create in-engine add-ons in LUA

The solution I came up with is very simple and has 3 components to it. Only one of these components is specific to my favored game engine, the remaining components are generic and can be modified to suite any project.

**Quick Links**

<a href="https://docs.google.com/spreadsheet/ccc?key=0AoGqxtUhFBJDdFRMU3c5RTNQSExiTnhYZFkxRUloU0E&usp=sharing" target="_blank">Google Spreadsheet Template</a>  
<a href="http://error454.github.io/csv-xml-i18n" target="_blank">Github Project (parser + ShiVa AI)</a>  
<a href="https://raw.githubusercontent.com/error454/csv-xml-i18n/master/shiva/I18N.ste" target="_blank">ShiVa ste</a>

## Overview

The workflow of this solution looks like this:

1.  Make a spreadsheet with all of your translations
2.  Export the spreadsheet as CSV
3.  Parse the CSV document into multiple XML documents, one for each language and file
4.  Read XML documents into the game engine and use the desired strings

## The Spreadsheet

Check out and make a copy of <a href="https://docs.google.com/spreadsheet/ccc?key=0AoGqxtUhFBJDdFRMU3c5RTNQSExiTnhYZFkxRUloU0E&usp=sharing" target="_blank">the spreadsheet template</a> to get started on your own translations.

The format of the spreadsheet is important. If it changes, the parser also needs to change. How the spreadsheet works is fairly intuitive and is probably easier done than said.

<img src='{{ site.url }}/assets/uploads/2014/04/localization.jpg' alt='The localization spreadsheet'>

This spreadsheet is automatically translating english to all other languages using the function:
<pre>GoogleTranslate( text, sourceLanguage, destinationLanguage)</pre>
Look at the formula bar for any translated cell and you can see that I've defaulted the source language to english while pulling the destination language from the language code field on the top row. We all know how good/bad auto-translation is, love it or hate it, you decide on the quality of your translations.

### Spreadsheet Instructions

*   Each filename entry must be preceded by an empty cell! If not, the parser will fail. Notice how the example has an empty cell before each filename. I'm fighting with the decision to make this text red and blinking to catch your attention.
*   To add a new file, create an entry for the filename in column A (make sure there is an empty cell above the filename!). In the example, there are two files defined (**MenuScreen** and **KeyBind**). The bold font is cosmetic only, only the text is preserved when exported to CSV.
*   To add an entry to a file, define the desired string identifier below the filename. Examples for **MenuScreen** are **title, start, options credits**. These identifiers must be unique for each file. For example, it's ok if every file has a **title** identifier, but there can't be two **title** identifiers in the same file.
*   To set a translation, move right from each identifier, filling in the translation for the current column as you go.

## The Parser

The parser is a small LUA script that lives in <a href="http://error454.github.io/csv-xml-i18n" target="_blank">the csv-xml-i18n repo</a>. The parser takes a CSV file as input. To generate a CSV file from Google Docs, select **File -> Download As -> Comma-separated values.**

If you don't have LUA installed on your system, you'll of course need that to run the code. You can then feed the parser your CSV file by passing the filename as the first argument:

<pre>lua csvToXml.lua test.csv</pre>

The result of running the script is that language files will be generated for each file and language defined. The naming convention is [filename]-[country code].xml.

## The Game Engine

Download my <a href="https://raw.githubusercontent.com/error454/csv-xml-i18n/master/shiva/I18N.ste" target="_blank">ShiVa AI</a> to easily implement localization in your game.

The I18N AI has the following features to make ShiVa localization a breeze:

### A Simple Interface for Fetching Strings

<pre>getString (Filename.identifier)</pre>

Examples:

<pre>log.message ( I18N.getString ( "MenuScreen.title" ) -- prints "Awesome Game!"
log.message ( I18N.getString ( "KeyBind.jump" ) )   -- prints "Jump"
log.message ( I18N.getString ( "KeyBind.attack" ) ) -- prints "Attack"</pre>

### Automatic XML Loading

The AI automatically loads the XML file matching the OS language. If your OS is set to french and you request a string from **MenuScreen**, it will first try to load **MenuScreen-fr.xml**. If it can't find that file, it will fall back to **MenuScreen-en.xml**. The fallback language is english.

### ShiVa Integration

To use the AI and XML files in your project, follow these simple steps:

1.  Add the XML files to your ShiVa project's resources/XML folder (be careful of editing these files in the shiva editor, it may destroy the UTF-8ness)
2.  Drag the XML files into the resources tab for your project
3.  Import the I18N AI and add it as the first UserAI
4.  Use the getString function!

There is also a cleanup function that can be called <pre>I18N.cleanUp()</pre>  that empties out the hash table used to store the XML file contents. Use it when you want to clean up strings that won't be used anymore.

## Conclusion

This system is simple and it works. Follow the rules and it will work for you too. If you use this system, please let me know in the comments!
