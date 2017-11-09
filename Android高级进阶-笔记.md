# Android 高级进阶

<h2 id="index">目录</h2>

- [基础篇](#part1)
  - [Android 触摸事件传递机制](#chap1.1)
  - [Android View 的绘制流程](#chap1.2)
  - [Android 动画机制](#chap1.3)
  - [Support Annotation Library 使用详解](#chap1.4)
  - [Percent Support Library 使用详解](#chap1.5)
  - [Design Support Library 使用详解](#chap1.6)
  - [Android Studio 中的 NDK 开发](#chap1.7)
  - [Gradle 必知必会](#chap1.8)
  - [通过 Gradle 打包发布函数库到 JCenter 和 Maven Central](#chap1.9)
  - [Builder 模式详解](#chap1.10)
  - [注解在 Android 中的应用](#chap1.11)
  - [ANR 产生的原因及其定位分析](#chap1.12)
  - [Android 异步处理技术](#chap1.13)
  - [Android 数据序列化方案研究](#chap1.14)
  - [Android WebView Java 和 JavaScript 交互详解](#chap1.15)
- [系统架构篇](#part2)
  - [MVP 模式及其在 Android 中的实践](#chap2.1)
  - [MVVM 模式及 Android DataBinding 实战](#chap2.2)
  - [观察者模式的拓展：事件总线](#chap2.3)
  - [书写简洁规范的代码](#chap2.4)
  - [基于开源项目搭建属于自己的技术堆栈](#chap2.5)
- [经验总结篇](#part3)
  - [64K 方法数限制原理与解决方案](#chap3.1)
  - [Android 插件框架机制研究与实践](#chap3.2)
  - [推送机制实现原理详解](#chap3.3)
  - [APP 瘦身经验总结](#chap3.4)
  - [Android Crash 日志收集原理与实践](#chap3.5)
- [新技术篇](#part4)
  - [函数式编程思想及其在 Android 中的应用](#chap4.1)
  - [依赖注入及其在 Android 中的应用](#chap4.2)
  - [Android 世界的 Swift：Kotlin 在 Android 中的应用](#chap4.3)
  - [React Native For Android 入门指南](#chap4.4)
  - [Android 在线热修复方案研究](#chap4.5)
  - [面向切面编程及其在 Android 中的应用](#chap4.6)
  - [基于 Facebook Buck 改造 Android 构建系统](#chap4.7)
- [性能优化篇](#part5)
  - [代码优化](#chap5.1)
  - [图片优化](#chap5.2)
  - [电量优化](#chap5.3)
  - [布局优化](#chap5.4)
  - [网络优化](#chap5.5)
- [移动安全篇](#part6)
  - [Android 混淆机制详解](#chap6.1)
  - [Android 反编译机制详解](#chap6.2)
  - [客户端敏感信息隐藏技术研究](#chap6.3)
  - [Android 加固技术研究](#chap6.4)
  - [Android 安全编码](#chap6.5)
- [工具篇](#part7)
  - [Android 调试工具 Facebook Stetho](#chap7.1)
  - [内存泄漏检测函数库 LeakCanary](#chap7.2)
  - [基于 Facebook Redex 实现 Android APK 的压缩和优化](#chap7.3)
  - [Android Studio 你所需要知道的功能](#chap7.4)
- [测试篇](#part8)
  - [Android 单元测试框架简介](#chap8.1)
  - [Android UI 自动化测试框架简介](#chap8.2)
  - [Android 静态代码分析实战](#chap8.3)
  - [基于 Jenkins + Gradle 搭建 Android 持续集成编译环境](#chap8.4)

------

<h2 id="part1">基础篇</h2>

<h3 id="chap1.1">Android 触摸事件传递机制</h3>

#### 触摸事件

触摸事件对应 `MotionEvent` 类。

- `ACTION_DOWN`

    用户手指的按下操作。一个按下操作标志着一次触摸事件的开始。

- `ACTION_MOVE`

    用户手指按压屏幕后松开前，如果移动距离超过一定阈值，会被判定为 `ACTION_MOVE` 操作。一般情况下，手指的轻微移动会触发一系列的移动事件。

- `ACTION_UP`

    用户手指离开屏幕的操作。一次抬起操作标志着一次触摸事件的结束。

在一次屏幕触摸操作中， `ACTION_DOWN` 和 `ACTION_UP` 这两个事件是必需的，而 `ACTION_MOVE` 视情况而定。

#### 事件传递

事件传递分为三个阶段：

- 分发 Dispatch

    事件分发对应 `dispatchTouchEvent(MotionEvent)` 方法。在该方法返回 `true` 表示事件被当前视图消费掉不再继续分发。方法返回 `false` 表示事件不再继续分发，传递回上层的 `onTouchEvent(MotionEvent)` 方法处理。方法返回 `super.dispatchTouchEvent(MotionEvent)` 表示继续分发该事件，并且如果当前视图是 `ViewGroup` 或其子类，会调用 `onInterceptTouchEvent(MotionEvent)` 方法判定是否拦截该事件。

- 拦截 Intercept

    事件拦截对应 `onInterceptTouchEvent(MotionEvent)` 方法。该方法只在 `ViewGroup` 及其子类中存在。该方法返回 `true` 表示拦截事件，不继续分发给子视图，传递给当前视图的 `onTouchEvent(MotionEvent)` 方法消费触摸事件。返回 `false` 或 `super.onInterceptTouchEvent(MotionEvent)` 表示不对事件进行拦截，继续传递给子视图。

- 消费 Consume

    事件消费对应 `onTouchEvent(MotionEvent)` 方法。该方法返回值为 `true` 表示当前视图消费该事件，事件将不会向上传递给上层视图。返回值为 `false` 或 `super.onTouchEvent(MotionEvent)` 表示当前视图不处理这个事件，事件传递给父视图的 `onTouchEvent(MotionEvent)` 方法进行处理。

<h3 id="chap1.2">Android View 的绘制流程</h3>
<h3 id="chap1.3">Android 动画机制</h3>
<h3 id="chap1.4">Support Annotation Library 使用详解</h3>
<h3 id="chap1.5">Percent Support Library 使用详解</h3>
<h3 id="chap1.6">Design Support Library 使用详解</h3>
<h3 id="chap1.7">Android Studio 中的 NDK 开发</h3>
<h3 id="chap1.8">Gradle 必知必会</h3>
<h3 id="chap1.9">通过 Gradle 打包发布函数库到 JCenter 和 Maven Central</h3>
<h3 id="chap1.10">Builder 模式详解</h3>
<h3 id="chap1.11">注解在 Android 中的应用</h3>
<h3 id="chap1.12">ANR 产生的原因及其定位分析</h3>
<h3 id="chap1.13">Android 异步处理技术</h3>
<h3 id="chap1.14">Android 数据序列化方案研究</h3>
<h3 id="chap1.15">Android WebView Java 和 JavaScript 交互详解</h3>

[↑ 目录](#index)

------

<h2 id="part2">系统架构篇</h2>

<h3 id="chap2.1">MVP 模式及其在 Android 中的实践</h3>
<h3 id="chap2.2">MVVM 模式及 Android DataBinding 实战</h3>
<h3 id="chap2.3">观察者模式的拓展：事件总线</h3>
<h3 id="chap2.4">书写简洁规范的代码</h3>
<h3 id="chap2.5">基于开源项目搭建属于自己的技术堆栈</h3>

[↑ 目录](#index)

------

<h2 id="part3">经验总结篇</h2>

<h3 id="chap3.1">64K 方法数限制原理与解决方案</h3>
<h3 id="chap3.2">Android 插件框架机制研究与实践</h3>
<h3 id="chap3.3">推送机制实现原理详解</h3>
<h3 id="chap3.4">APP 瘦身经验总结</h3>
<h3 id="chap3.5">Android Crash 日志收集原理与实践</h3>

[↑ 目录](#index)

------

<h2 id="part4">新技术篇</h2>

<h3 id="chap4.1">函数式编程思想及其在 Android 中的应用</h3>
<h3 id="chap4.2">依赖注入及其在 Android 中的应用</h3>
<h3 id="chap4.3">Android 世界的 Swift：Kotlin 在 Android 中的应用</h3>
<h3 id="chap4.4">React Native For Android 入门指南</h3>
<h3 id="chap4.5">Android 在线热修复方案研究</h3>
<h3 id="chap4.6">面向切面编程及其在 Android 中的应用</h3>

[↑ 目录](#index)

------

<h2 id="part5">性能优化篇</h2>

<h3 id="chap5.1">代码优化</h3>
<h3 id="chap5.2">图片优化</h3>
<h3 id="chap5.3">电量优化</h3>
<h3 id="chap5.4">布局优化</h3>
<h3 id="chap5.5">网络优化</h3>

[↑ 目录](#index)

------

<h2 id="part6">移动安全篇</h2>

<h3 id="chap6.1">Android 混淆机制详解</h3>
<h3 id="chap6.2">Android 反编译机制详解</h3>
<h3 id="chap6.3">客户端敏感信息隐藏技术研究</h3>
<h3 id="chap6.4">Android 加固技术研究</h3>
<h3 id="chap6.5">Android 安全编码</h3>

[↑ 目录](#index)

------

<h2 id="part7">工具篇</h2>

<h3 id="chap7.1">Android 调试工具 Facebook Stetho</h3>
<h3 id="chap7.2">内存泄漏检测函数库 LeakCanary</h3>
<h3 id="chap7.3">基于 Facebook Redex 实现 Android APK 的压缩和优化</h3>
<h3 id="chap7.4">Android Studio 你所需要知道的功能</h3>

[↑ 目录](#index)

------

<h2 id="part8">测试篇</h2>

<h3 id="chap8.1">Android 单元测试框架简介</h3>
<h3 id="chap8.2">Android UI 自动化测试框架简介</h3>
<h3 id="chap8.3">Android 静态代码分析实战</h3>
<h3 id="chap8.4">基于 Jenkins + Gradle 搭建 Android 持续集成编译环境</h3>

[↑ 目录](#index)

------
