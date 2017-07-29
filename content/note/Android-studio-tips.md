+++
title = "Android studio tips"
draft = false
date = "2017-03-11T16:08:07+08:00"
tags = [ "Android Studio", "tips" ]
categories = [ "Note" ]
+++

## Share module between projects  
Add this to `settings.gradle` file at the project root: 

    include ':libraryName' 
    project(':libraryName').projectDir=new File('/path/to/library')

where the path you specify in the second line is the path to the directory containing the library's `build.gradle` file. The path can be relative or absolute.

## Avoid so file can not find problem  
If library's **jniLibs** contians **armeabi-v7a** folder, but main module's **jniLibs** contains not, the app will crash because of so file could not found.

## Problems after change sdk location  
After change **sdk loaction**, android studio always fetching documentation from network even you have downloaded the documentation with sdk tools, and no longer auto attach the support library source code. To fix these problems, you should close the project and reopen it(not use Open Recent).

## Incompatible type error occurs when compile  
The compiler error output make no sense(An incompatible type error occurs at an import class line), maybe the real position of error is in `BuildConfig`. 

## Define `String` BuildConfig field in gradle  
When define a `String` BuildConfig field in build.gradle file, make sure the field value is surrounded with quote, even the gradle variable type is `String`, otherwise the field type will wrong.
```dsl
def fieldValue = "value"
...
buildConfigField "String", "FIELD_NAME", "\"${fieldValue}\""
```
