+++
draft = false
date = "2017-04-29T18:08:07+08:00"
title = "Android tips"
tags = [ "Android", "tips" ]
categories = [ "Note" ]
+++

* 如果要在低版本系统上使用高版本系统才加入的类，可以在app中加入同名类，在高版本系统上安装时这个同名类会删除，还是使用系统的类，在低版本系统则使用在app中加入的类。

* 在不同的`PendingIntent`的`Intent`中设置extra，在处理`Intent`的地方获取的extra始终相同，可以通过设置不同RequestCode来区分不同的`PendingIntent`。

* 对于replace fragment，通过代码和通过配置文件加入最初的fragment这两种方式之间是有区别的，当通过代码的加入最初的fragment时，replace一个新的fragment后再回退，会调最初的fragment的onCreateView方法，说明replace时最初的fragment被销毁了，而通过配置文件的方法则不会调用onCreateView方法。

* 在xml中加入fragment时键盘会自动弹出，通过代码加入时不会。

* `AsyncTask`在3.0以前是并行执行的，3.0及以后顺序执行的，`AsyncTask`是通过静态的`Handler`成员变量将结果post到主线程中执行的，所以要确保加载`AsyncTask`类是在主线程中执行的，在4.1及以后系统已经确保的这一点。

* 如果自己decode ninepatch图片，要先compile ninepatch图片。

* android ListView Adapter's getView method sometimes provide a wrong type convertView. [stackoverflow](http://stackoverflow.com/questions/12018997/why-does-getview-return-wrong-convertview-objects-on-separatedlistadapter)

* 市场解锁、gaeproxy等软件会修改运营商信息。

* cookie里包含有特殊字符会导致请求失败。

* Bitmap.createBitmap在低内存时在有些手机上可能返回null，而不是抛OutofMemoryError。

* 对于apk压缩包的操作应使用android自带的aapt，避免直接使用zip，直接使用zip会将apk里面的raw资源压缩，导致有些rom下读取raw文件失败。

* 如果不给`WebView`设置`WebViewClient`，在有些手机上会自动跳到系统浏览器。

* 在Androd 7.0上如果使用`FLAG_ACTIVITY_REORDER_TO_FRONT`来启动activity，快速连续点击不断启动activity时，可能会回到桌面。