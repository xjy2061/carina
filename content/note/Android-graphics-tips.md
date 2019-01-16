+++
date = "2018-09-26T10:50:10+08:00"
title = "Android graphics tips"
draft = false
tags = [ "Android", "graphics", "tips" ]
categories = [ "Note" ]
+++

* 如果给`Paint`同时设置了shader和color，会优先使用shader的颜色，例如`drawText`时，文字颜色是shader的颜色，而非color所指定的颜色。

* 虽然官方文档表明开启硬件加速时，只要`ComposeShader`里包含的shader是不同的类型且不是`ComposeShader`，那么在api 28以下也可以使用`ComposeShader`，但实际测试结果是只有关闭硬件加速`ComposeShader`才起作用。

* 如果给`Paint`设置了**PathEffect**，path的FillType是**EVEN_ODD**时会失效。
