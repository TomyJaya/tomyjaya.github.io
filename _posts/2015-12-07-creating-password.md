---
layout:     post
title:      "Creating Password"
subtitle:   "Simple Guidelines When Creating A System Id Password"
date:       2015-12-07 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-06.jpg"
tags:
- til
- guide
---

# Today I Learnt (TIL)

Do not use fancy/ reserved/ escape characters in your system ID password!  

# Why?

I stumbled upon a case whereby a system Id has got multiple special characters in it today. Encryption and serialization went haywire and I spent so much time debugging it so I thought of giving guidelines: 

1. Don't use *quotes* nor *double quotes* as they are mostly used to enclose string literals
2. Don't use *backslashes* as they are usually used as an escape sequence 
3. Don't use *colons* as they are commonly used as a key value separator (e.g. in JSON)

I know sometimes we want to make password hard to crack by adding special characters, but I think the above should be *strictly off limits* as they might cause you serious time wastage when trying to serialize or encrypt them. 
