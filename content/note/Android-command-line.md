+++
date = "2017-04-29T18:54:21+08:00"
title = "Android command line"
draft = false
tags = [ "Android", "command line" ]
categories = [ "Note" ]
+++

* **Show current tasks**  
adb shell dumpsys activity activites  

* **Show screen infos**  
adb shell dumpsys window

* **Show gfxinfo**  
adb shell dumpsys gfxinfo \<PACKAGE_NAME\>