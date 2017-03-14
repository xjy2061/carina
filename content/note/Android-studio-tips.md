+++
title = "Android studio tips"
draft = false
date = "2017-03-11T16:08:07+08:00"
tags = [ "Android Studio", "tips" ]
categories = [ "Note" ]
+++

#### Share module between projects
Add this to `settings.gradle` file at the project root: 

    include ':libraryName' 
    project(':libraryName').projectDir=new File('/path/to/library')

where the path you specify in the second line is the path to the directory containing the library's `build.gradle` file. The path can be relative or absolute.

#### Avoid so file can not find problem
If library's **jniLibs** contians **armeabi-v7a** folder, but main module's **jniLibs** contains not, the app will crash because of so file could not found.