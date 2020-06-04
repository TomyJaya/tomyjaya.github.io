---
layout:     post
title:      "The Java BigDecimal Constructor"
subtitle:   "Which BigDecimal Constructor Should You Use? "
date:       2015-12-16 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-06.jpg"
tags:
- til
- java
---

# Today I Learnt (TIL)

To always use the java.math.BigDecimal *constructor which accepts String argument*. 

# Why?

The constructor accepting double might behave 'unexpectedly' as it retains the floating point precision of the double input. 


# Resources

* [The Curious Schemer - The Evil BigDecimal Constructor](http://rayfd.me/2007/04/26/the-evil-bigdecimal-constructor/)
* [This StackOverflow entry](http://stackoverflow.com/questions/11368496/java-bigdecimal-bugs-with-string-constructor-to-rounding-with-round-half-up) 
* [Video of a Java Puzzler by Josh Bloch on Google I/O 2011](https://www.youtube.com/watch?v=wbp-3BJWsU8&feature=player_detailpage#t=243s)
