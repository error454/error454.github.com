---
title: Quicksort for ShiVa3D
author: error454
layout: post
permalink: /2014/02/10/quicksort-for-shiva3d/
categories:
  - Shiva 3D
tags:
  - quicksort
  - shiva
---
If you ever need to sort a table of numbers in shiva, I adapted this <a href="http://rosettacode.org/wiki/Sorting_algorithms/Quicksort#Lua" target="_blank">Rosetta Code entry</a> for shiva.

{% gist error454/8928424 %}



The following code:

<pre>local tTest = table.newInstance ( )
table.add ( tTest, 1)
table.add ( tTest, 9)
table.add ( tTest, 7)
table.add ( tTest, 3)
table.add ( tTest, 5)
table.add ( tTest, 10)
table.add ( tTest, 6)
table.add ( tTest, -5)
table.add ( tTest, -9)
table.add ( tTest, -15)
table.add ( tTest, 33)
table.add ( tTest, 4)
table.add ( tTest, 8)

this.quicksort (tTest)
for i = 0, table.getSize ( tTest) - 1 do
    log.message ( table.getAt ( tTest, i ) )
end</pre>

Produces the output:

<pre>-15
-9
-5
1
3
4
5
6
7
8
9
10
33</pre>
