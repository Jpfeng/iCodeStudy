# Android 群英传

<h2 id="index">目录</h2>

- [Android 体系与系统架构](#chap1)
- [Android 开发工具](#chap2)

------

<h2 id="chap1">Android 体系与系统架构</h2>

Android 系统架构： Linux内核层 → 库和运行时 → Framework层 → 应用层

Android 四大组件： `Activity` `Content Provider` `Service` `BroadCast Reciever`

信息传递的载体： `Intent`

Context 实现类： `Activity` `Service` `Application`

[Android 系统源代码](http://androidxref.com/)

- `/system/app/`  一些系统的应用程序。
- `/system/bin/`  Linux 自带的组件。
- `/system/build.prop`  系统的属性信息。
- `/system/fonts/`  存放系统字体。
- `/system/framework/`  系统的核心文件、框架层。
- `/system/lib/`  存放几乎所有的共享库 `.so` 文件。
- `/system/media/`  保存系统提示音、系统铃声。
- `/system/usr/`  保存用户的配置文件。
- `/data/app/`  包含了用户安装或升级的应用程序。
- `/data/data/`  应用程序的数据信息、文件信息、数据库信息等。
- `/data/system/`  包含了手机的各项系统信息。
- `/data/misc/` 保存了大部分的 Wi-Fi 、 VPN 信息。

[↑ 目录](#index)

------

<h2 id="chap2">Android 开发工具</h2>

### Android Studio

取代 Eclipse 。

### ADB 命令

Android Debug Bridge 位于 SDK 的 `platform-tools` 目录下。

- `adb version`  显示当前版本。
- `adb shell`  使用 Linux Shell 命令。
- `adb install PACKAGE`  安装应用程序。
- `adb push LOCAL... REMOTE`  将文件写入到设备。
- `adb pull REMOTE... LOCAL`  从设备获取文件。
- `adb reboot`  重启设备。

### 模拟器

[Genymotion](http://www.genymotion.net/)

[↑ 目录](#index)

------
