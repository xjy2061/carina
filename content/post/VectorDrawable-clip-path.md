+++
date = "2017-03-14T11:54:56+08:00"
title = "VectorDrawable clip path"
draft = false
tags = [ "Android", "VectorDrawable", "clip-path" ]
categories = [ "Post" ]
+++

# Function
**clip path**限制在画布上绘制的区域，在**clip path**所指定的区域外的位置不会被绘制。

我们通过下面的心形图标来展示**clip path**的效果：
```xml
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="56dp"
    android:height="56dp"
    android:viewportHeight="56"
    android:viewportWidth="56">

  <path
      android:fillColor="@android:color/black"
      android:pathData="M28,39 L26.405,37.5667575 C20.74,32.4713896 17,29.1089918 17,24.9945504 C17,21.6321526 19.6565,19 23.05,19 C24.964,19 26.801,19.8828338 28,21.2724796 C29.199,19.8828338 31.036,19 32.95,19 C36.3435,19 39,21.6321526 39,24.9945504 C39,29.1089918 35.26,32.4713896 29.595,37.5667575 L28,39 L28,39 Z"/>

</vector>
```
![heart](heart.png)  
<font size=3>**Figure1** 心形图标</font>

下面加入`<clip-path>`将绘制区域限制在[0, 0, 56, 28]的矩形区域内：
```xml
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="56dp"
    android:height="56dp"
    android:viewportHeight="56"
    android:viewportWidth="56">

  <clip-path
      android:name="clip"
      android:pathData="M0 0 L56 0 L56 28 L0 28 Z"/>

  <path
      android:fillColor="@android:color/black"
      android:pathData="M28,39 L26.405,37.5667575 C20.74,32.4713896 17,29.1089918 17,24.9945504 C17,21.6321526 19.6565,19 23.05,19 C24.964,19 26.801,19.8828338 28,21.2724796 C29.199,19.8828338 31.036,19 32.95,19 C36.3435,19 39,21.6321526 39,24.9945504 C39,29.1089918 35.26,32.4713896 29.595,37.5667575 L28,39 L28,39 Z"/>

</vector>
```
![clip-heart](clip-heart.png)  
<font size=3>**Figure2** clip后的心形图标</font>

# Application
可通过改变`<clip-path>`的`pathData`值来实现一些动画效果。

下面通过**clip path**来实现填充心形图标的效果：
```xml
<animated-vector
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:aapt="http://schemas.android.com/aapt"
  android:drawable="@drawable/vd_trimclip_heart_full">

  <target android:name="clip">
    <aapt:attr name="android:animation">
      <objectAnimator
        android:duration="@android:integer/config_mediumAnimTime"
        android:interpolator="@android:interpolator/fast_out_slow_in"
        android:propertyName="pathData"
        android:valueFrom="M18 37 L38 37 L38 37 L18 37 Z"
        android:valueTo="M0 0 L56 0 L56 56 L0 56 Z"
        android:valueType="pathType"/>
    </aapt:attr>
  </target>

</animated-vector>
```
vd\_trimclip\_heart_full.xml：
```xml
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="56dp"
    android:height="56dp"
    android:tint="?attr/colorControlNormal"
    android:viewportHeight="56"
    android:viewportWidth="56">

  <path
      android:pathData="M28.7194825,38.2960196 L25.6688233,35.5523072 C21.6213379,31.793335 18.0159912,28.8906251 18.0159912,24.845337 C18.0159912,21.5882568 20.6305373,19.9651315 23.6337891,19.9651442 C24.9985352,19.96515 26.7993165,21.180664 28.643982,23.1297608"
      android:strokeColor="@android:color/white"
      android:strokeWidth="2"/>

  <path
      android:pathData="M27.2310342,38.2944327 L30.7645263,35.2004395 C34.8336142,31.2354829 37.751709,29.1181308 38.0040283,25.0838624 C38.1681943,22.4590544 35.7730552,20.034668 33.3791503,20.034668 C30.4320068,20.034668 29.671997,21.0472412 27.2310342,23.1328126"
      android:strokeColor="@android:color/white"
      android:strokeWidth="2"/>

  <group android:name="filled">
    <clip-path
        android:name="clip"
        android:pathData="M18 37 L38 37 L38 37 L18 37 Z"/>
    <path
        android:fillColor="@android:color/white"
        android:pathData="M28,39 L26.405,37.5667575 C20.74,32.4713896 17,29.1089918 17,24.9945504 C17,21.6321526 19.6565,19 23.05,19 C24.964,19 26.801,19.8828338 28,21.2724796 C29.199,19.8828338 31.036,19 32.95,19 C36.3435,19 39,21.6321526 39,24.9945504 C39,29.1089918 35.26,32.4713896 29.595,37.5667575 L28,39 L28,39 Z"/>
  </group>

</vector>
```
![fill-heart](fill-heart.gif)  
<font size=3>**Figure3** 心形图标填充效果</font>