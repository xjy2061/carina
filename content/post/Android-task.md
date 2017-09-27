+++
date = "2017-09-24T14:44:33+08:00"
title = "Android task"
draft = false
tags = ["Android", "Task"]
categories = [ "Post" ]
+++

## `singleTask` and `FLAG_ACTIVITY_NEW_TASK`
根据下面引用的官方文档的说法，`singleTask`和`FLAG_ACTIVITY_NEW_TASK`的行为是一致的，但是根据实际测试结果，二者的行为是有差别的。

> This produces the same behavior as the "singleTask" launchMode value, discussed in the previous section.

二者在决定activity属于哪个task上是一致的，都根据`taskAffinity`（如果没设，默认为包名）来决定activity属于哪个task，如果当前存在一个名称跟`taskAffinity`相同的task，则将activity置于已有的这个task，否则创建一个新task，并将activity作为这个task的根activity。

二者在启动activity的行为上是不一致的，如果activity所属的task已存在且task中已存在这个activity的实例，二者都会将整个task移到前台，但`singleTask`会关闭这个activity上面的所有activity，并调用这个activity的`onNewIntent`方法，而对于`FLAG_ACTIVITY_NEW_TASK`，如果启动的activity是这个task的根activity，不做任何事情，否则会创建一个新的activity。

## Exported activity
启动其他app的exported activity时，会忽略`launchMode`，此时只能用`FLAG_ACTIVITY_NEW_TASK`来创建新task。

## `startActivityForResult`
调用`startActivityForResult`来启动activity时，如果使用了`FLAG_ACTIVITY_NEW_TASK`，启动activity后会立即收到`onActivityResult`回调，且finish启动的activity后不会再收到`onActivityResult`回调，而使用`singleTask`和`singleInstance`则是正常的行为。



