---
layout:     post
title:      "Formatting JSON in command line"
subtitle:   "Python json.tool to the rescue!"
date:       2016-01-22 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-06.jpg"
tags:
- til
- cli
---

# Today I Learnt (TIL)
To format/ pretty-print a one-line JSON file, you can use python json.tool module. 

For example: 

```bash
python -m json.tool instafeed.json > instafeed_formatted.json
```
 
Piping can also work: 

``` bash
echo {name: "Tomy", gender: "M"} | python -m json.tool 
```

If you're an **Amazon Linux** or **Mac OS** user, this command is very handy as both of them come with **Python 2.6+ installed by default**. 

Hence, it's recommended to **alias** it: 

``` bash
alias prettyjson='python -m json.tool'
```

Speaking of aliases, perhaps I should share my alias list (in Mac and Amazon EC2) in a future blog post. Please remind me in case I forget. :)

# Source: 
[http://stackoverflow.com/questions/352098/how-can-i-pretty-print-json](http://stackoverflow.com/questions/352098/how-can-i-pretty-print-json)

