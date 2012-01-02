---
layout: post
title: Solution for System Time Error When Compiling Packages
categories:
- linux
---

I was trying to recompile Nginx today, But I got a very wierd error that I couldn't pass it.

This is the error information,

    configure: error: newly created file is older than distributed files!
    Check your system clock
    
It seems that the file I created is lolder that the distributed files, then I check the system clock,
I found the timezone was wrong, so I reset the timezone with:

    sudo dpkg-reconfigure tzdata
    
Choose your current timezone, and it will update your system clock.

Awesome, it works.