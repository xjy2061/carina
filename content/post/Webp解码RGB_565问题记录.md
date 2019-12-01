+++
title = "Webp解码RGB_565问题记录"
draft = false
date = "2019-11-30T20:55:09+08:00"
tags = [ "Android", "webp", "RGB_565" ]
categories = [ "Post" ]
+++

最近遇到自定义webp解码使用rgb565时出现颜色错乱的问题，查了比较长时间，总结一下过程。

开始时怀疑代码逻辑有问题，于是写demo测试native代码逻辑，尝试修改`Bitmap`的一些属性，确认代码逻辑正确。

然后考虑到用系统解码api和官方ndk sample中的webp解码示例都正常，怀疑是不是对**libwebp**的调用方式有问题，于是去看源码。先从简单的ndk sample开始看，发现调用方式是一样的，然后看了系统webp解码部分源码，发现系统使用的解码函数不一样，尝试换成跟系统一样的函数，问题依然存在。由于官方示例并没有将图片解码成`Bitmap`，所以接着仔细看系统源码的解码过程，看是不是在某些细节里有特殊处理，最后没有发现有特殊的地方。

看了几天源码无果后，最后想到是不是解码得到的`Bitmap`的底层像素数据不一样，而不是对`Bitmap`的设置不一样，于是写测试代码获取`Bitmap`的像素字节数组，对2种方式解码得到的`Bitmap`的像素数据进行比较，发现同一像素的2个字节的位置刚好相反。
````
[74, 105, -125, -18, ...]

[105, 74, -18, -125, ...]
````
到这里终于有点眉目了，后面就去找导致位置相反的原因，然后在**libwebp**源码定义像素格式的地方发现了一段注释，感觉很可能跟`WEBP_SWAP_16BITS_CSP`有关，但在代码中没有找到这个变量。
````c++
// ...
// RGB-565: [r4 r3 r2 r1 r0 g5 g4 g3], [g2 g1 g0 b4 b3 b2 b1 b0], ...
// In the case WEBP_SWAP_16BITS_CSP is defined, the bytes are swapped for
// these two modes:
// RGBA-4444: [b3 b2 b1 b0 a3 a2 a1 a0], [r3 r2 r1 r0 g3 g2 g1 g0], ...
// RGB-565: [g2 g1 g0 b4 b3 b2 b1 b0], [r4 r3 r2 r1 r0 g5 g4 g3], ...

typedef enum WEBP_CSP_MODE {
  // ...
  MODE_RGB_565 = 6,
  // ...
} WEBP_CSP_MODE;
````
最后怀疑`WEBP_SWAP_16BITS_CSP`是在编译时指定的，后来终于在官方示例工程的**CMakeLists**文件和系统源码的**libwebp**编译配置中找到了它的踪迹。
````cmake
# CMakeList.txt
if(WEBP_ENABLE_SWAP_16BIT_CSP)
  add_definitions(-DWEBP_SWAP_16BIT_CSP=1)
endif()
````
````mk
# Android.bp
cflags: [
    "-O2",
    "-DANDROID",
    "-DWEBP_SWAP_16BIT_CSP",
    "-Wall",
    "-Werror",
 ],
````
到这里问题原因已基本查明，然后修改图片库**libwebp**编译配置，打包验证，使用`RGB_565`解码的图片正常显示了。
````mk
WEBP_CFLAGS := -Wall -DANDROID -DHAVE_MALLOC_H -DHAVE_PTHREAD -DWEBP_USE_THREAD -DWEBP_FORCE_ALIGNED -DWEBP_SWAP_16BIT_CSP
````