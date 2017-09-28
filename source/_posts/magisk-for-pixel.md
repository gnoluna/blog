---
title: "千辛万苦等来了 Magisk 对 Pixel(XL) 的官方支持"
date: 2017-09-28 17:13:40
tags: [Android, Magisk, Pixel]
catagories: tweeks
---

自从有了折腾手机的念头，我就从未放弃过给自己的 Pixel sailfish 装上 [Magisk](https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445)。毕竟 Xposed 的作者已经以一副要弃坑的姿势搁置这个目前仍相对主流的框架，为了能在抵达欧陆之后玩上**小畜生**以及其他定制需求, Magisk 成了最吸引人的 systemless root 解决方案。然而 Pixel 的 A/B partition 造成了各种不兼容。众 Pixel(XL) 用户只能依赖[非官方支持](https://forum.xda-developers.com/apps/magisk/unofficial-google-pixel-family-support-t3639262) (此处感谢 goodwin_c)。这在 Android 8.0 之前还都不是事，Oreo 一出我们就只能巴望着 topjohnwu 的更新（为此我还设置了 twitter 提醒 orz）。

好在 topjohnwu 是个勤劳的开发者，度假回来马上更新了 [v14.1](https://forum.xda-developers.com/apps/magisk/beta-magisk-v13-0-0980cb6-t3618589)，感恩。

目前为止在我的 Pixel sailfish Android 8.0 上没有问题，安装过程中也没出什么幺蛾子。在此记录下安装过程。

**DISCLAIMER: 虽然几乎不可能刷砖，但还是得说一句 Do at your own risk.**

### 前期准备

-	下载 [Magisk v14.1](https://forum.xda-developers.com/attachment.php?s=b12bee961638472dc0099d6e042e0cb4&attachmentid=4285325&d=1506549792)
-	下载 [twrp-3.1.1-0-fastboot-sailfish.img](https://dl.twrp.me/sailfish/twrp-3.1.1-0-fastboot-sailfish.img.html) (已经刷过 TWRP recovery 的可以忽略这项)。然后重命名为 twrp.img，出于方便键入的考虑。
-	下载 [Magisk Manager v5.3.5](https://github.com/topjohnwu/MagiskManager/releases/download/v5.3.5/MagiskManager-v5.3.5.apk)
- 电脑上安装 android-platform-tools (会用到 adb 和 fastboot)
- 手机已解锁 OEM
- 手机上设置开启 PIN 或者 Pattern 密码（要不然进入 recovery 之后文件夹会被加密完全找不到想要的文件）

**特别提示** 如果你之前折腾过，要刷回 Pixel 的原厂 boot.img 或者刷入 [Magisk Uninstaller](https://forum.xda-developers.com/attachment.php?attachmentid=4285327&d=1506549946)

### 安装过程

1. 手机链接电脑，将 **Magisk-v14.1(1410).zip** 和 **MagiskManager-v5.3.5.apk** 放入存储中
2. 开启 usb 调试
3. `adb devices` 确认手机被识别到
4. `adb reboot bootloader` 或者关机长按**电源+音量-**进入 bootloader 模式。
3. `fastboot devices` 确认手机被识别到
5. `fastboot boot /path/to/twrp.img` 
6. 输入密码进入 twrp recovery，选择 install，找到并选中 **Magisk-v14.1(1410).zip**，swipe 刷入。完成后重启
7. 重启后安装 **MagiskManager-v5.3.5.apk**
8. 看到此界面说明您已经安装成功

<img src="/uploads/magisk_screenshot.png" width=50% height=50%/>


### 新特性

新版本发布前一周 Magisk 作者 topjohnwu 就[在 Twitter 上](https://twitter.com/topjohnwu/status/908542227143471104)说针对 Pixel A/B 分区会有新特性。那就是，Magisk 和 OTA 更新兼得。无法 OTA 更新可以说是 root 之后最不方便的一件事，现在解决了。

>	Pixel Exclusive Features
Surprise surprise! Thanks to the A/B partition of Pixel devices, we can actually receive and apply OTAs seamlessly using the stock OTA mechanism and preserve Magisk at the same time! 

简单说就是在安装 Magisk 的时候它已经备份了原厂 boot 镜像。在需要 OTA 更新时还原到原厂 boot 接收更新。安装完成后再将 Magisk 装回去 _Install to Second Slot(After OTA)_。重启之后 Magisk 不会被卸载，因为在启动时 Magisk 已经自动给新版本的 boot 打上补丁。

作者也在文档中添加了 OTA 更新相关的[指南](https://github.com/topjohnwu/Magisk/blob/master/docs/tips.md#ota-installation-tips)。亲妈一样的人文关怀。

Magisk 是好东西哎~
