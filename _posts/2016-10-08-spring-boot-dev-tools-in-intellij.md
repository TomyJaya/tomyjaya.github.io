---
layout:     post
title:      "Setting Up Spring Boot Dev Tools in IntelliJ IDEA"
subtitle:   "For Automatic Restart and Productivity Boost"
date:       2016-10-08 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-06.jpg"
tags:
- intellij-idea
- spring-boot
---

# The Problem
When you migrate from a dedicated container to use embedded-container-based solution like Spring Boot, you might find that automatic class re-loading is *not* readily available anymore. That is, you have to restart your application if you make a change on any Java classes.  
 

# The Solution
Luckily, Spring team feels your pain and came up with *Spring Boot Dev Tools*. In essence, it's a dependency which automatically restarts your Spring Boot Application when it detects any changes in your Classpath. The restart is much faster than the initial one as it intelligently finds out the only changed components to reload. 

You can just add the below dependency in your pom.xml if you're using Maven: 

~~~ xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
~~~

However, if you use *IntelliJ IDEA*, you will realize that adding the above it's just not enough. This is because unlike Eclipse, you need to explicitly tell IntelliJ IDEA to "Make The Project" for it to build to the target classpath. 

You obviously don't want to do this. So, the best way is to use Macros to invoke `"Make Project"` upon a `"Save All"` invocation (`Cmd`+`S` or `Ctrl`+`S`). 

To do that, follow the below simple steps: 

1. `Edit` - `Macros` - `Start Macro Recording`
2. `File` - `Save All`
3. `Build` - `Make Project`
4. `Edit` - `Macros` - `Stop Macro Recording` - You can save the macro name as "Save & Make Project"
5. `Preferences` - `Keymap` - `Copy` - You can name the new keymap as "Spring Boot Application"
6. Go down to expand the `macros` directory to find your newly macro (i.e. "Save & Make Project").
7. Double click to `Add Keyboard Shortcut` and press `Cmd`+`S` if you use Mac and `Ctrl`+`S` if you use Windows. 

That's it! Try to run your Spring Boot application and make a change in your API. Invoke `Cmd`+`S` or `Ctrl`+`S` and voila! You should see in your run console that Spring automatically restarts! 

You're welcome! :)

# References
1. [Official Spring Boot Docs on Boot Dev Tools](http://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html)
2. [StackOverflow entry to set up make project automatically](http://stackoverflow.com/questions/14635602/intellij-make-project-automatically-woes)