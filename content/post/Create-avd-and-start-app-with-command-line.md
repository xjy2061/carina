+++
date = "2017-03-21T12:23:32+08:00"
title = "Create avd and start app with command line"
draft = false
tags = [ "Android", "emulator", "command line" ]
categories = [ "Post" ]
+++

# Install sdk, create or delete avd

```shell
android list sdk -a
android update sdk -a -u --filter [id1, id2]

android list targets
android create avd -n 4.1-x86 -t 1 -b x86 -c 512M

android delete avd -n 4.1-x86
```

# Start emulator and launch app

```shell
android list avd
emulator64-x86 -avd 4.1-x86 -no-window -no-boot-anim -qemu -m 1024 -enable-kvm &

adb shell am start fm.xiami.main/fm.xiami.bmamba.activity.StartActivity

adb forward tcp:11874 tcp:11873
```

on 64-bits machines you may need install ia32-libs to run adb:
```shell
sudo apt-get install ia32-libs
```