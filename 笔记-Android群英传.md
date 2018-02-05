# Android 群英传

<h2 id="index">目录</h2>

- [第一章 Android 体系与系统架构](#chap1)
- [第二章 Android 开发工具](#chap2)
- [第三章 Android 控件](#chap3)
- [第四章 ListView 使用](#chap4)
- [第五章 Android Scroll 分析](#chap5)
- [第六章 Android 绘图机制](#chap6)
- [第七章 Android 动画机制](#chap7)
- [第八章 Activity 与 Activity 调用栈](#chap8)
- [第九章 系统信息与安全机制](#chap9)
- [第十章 Android 性能优化](#chap10)
- [第十一章 搭建云端服务器](#chap11)
- [第十二章 Android Lollipop 新特性详解](#chap12)
- [第十三章 Android 实例提高](#chap13)

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

<h2 id="chap3">Android 控件</h2>

### Android 控件架构

`View` 与 `ViewGroup` 。 `ViewGroup` 继承自 `View` ， `ViewGroup` 控件可以作为父控件包含多个 `View` 控件，并管理其包含的 `View` 控件。整个界面上的控件形成了一个树形结构，称为控件树。上层控件负责下层子控件的测量与绘制，并传递交互事件。每棵控件树的顶部，都有一个 `ViewParent` 对象，是整棵树的控制核心，所有的交互管理事件都由它来统一调度和分配。

在 Activity 中使用 `setContentView(int)` 方法设置一个布局。每个 Activity 都包含一个 `Window` 对象，通常由 `PhoneWindow` 来实现。 `PhoneWindow` 将一个 `DecorView` 设置为整个应用窗口的根 View 。 `DecorView` 封装了窗口操作的通用方法，这里面的所有 View 的监听事件，都通过 `WindowManagerService` 来进行接收，并通过 `Activity` 对象回调相应的监听器。在显示上， `DecorView` 将屏幕分成两部分。一个是 `TitleView` 。另一个是 `ContentView` 。它是一个 ID 为 `android.R.id.content` 的 `Framelayout` ， Activity 布局就是设置在其中。如果用户通过设置 `requestWindowFeature(Window.FEATURE_NO_TITLE)` 设置全屏显示，视图树中的布局就只有 `ContentView` 了。

当程序在 `onCreate(Bundle)` 方法中调用 `setContentView(int)` 方法后， `ActivityManagerService` 会回调 `onResume()` 方法。此时系统会把整个 `DecorView` 添加到 `PhoneWindow` 中，并让其显示出来，从而最终完成界面的绘制。

### View 测量

Android 系统在绘制 View 前，必须对 View 进行测量。这个过程在 `onMeasure(int, int)` 方法中进行。系统提供了 `MeasureSpec` 类测量 View 。 `MeasureSpec` 是一个 32 位的 `int` 值，其中高 2 位为测量的模式，低 30 位为测量的大小。在计算中使用位运算的原因是为了提高并优化效率。

测量的模式可以为以下三种：

- `EXACTLY`

    精确值模式。当我们将控件的 `layout_width` 属性或 `layout_height` 属性指定为具体数值或 `match_parent` 属性时，系统使用 `EXACTLY` 模式。

- `AT_MOST`

    最大值模式。当控件的 `layout_width` 属性或 `layout_height` 属性指定为 `wrap_content` 时，控件大小一般随着控件的子空间或内容的变化而变化。此时控件的尺寸不超过父控件允许的最大尺寸即可。

- `UNSPECIFIED`

    这个属性不指定其大小测量模式， View 想多大就多大。通常情况下在绘制自定义 View 时会使用。

`View` 类默认的 `onMeasure(int, int)` 方法只支持 `EXACTLY`模式。如果在自定义控件时不重写 `onMeasure(int, int)` 方法，就只能使用 `EXACTLY` 模式。控件可以响应指定的具体宽高值或是 `match_parent` 属性。如果让自定义 View 支持 `wrap_content` 属性，就必须重写 `onMeasure()` 方法指定 `wrap_content` 时的大小。

在重写 `onMeasure(int, int)` 方法时，先通过 `MeasureSpec.getMode(int)`  和 `MeasureSpec.getSize(int)` 方法从 `MeasureSpec` 对象中提取出测量模式和大小。接下来，通过判断测量的模式，给出不同的测量值。当模式为 `EXACTLY` 时，直接使用指定的大小即可。当模式为其他两种模式时，需要给定一个默认的大小。特别地，如果指定 `wrap_content` 属性，即 `AT_MOST` 模式，则需要取出指定大小与测量大小中最小值来作为最后的测量值。最终，调用 `setMeasuredDimension(int, int)` 方法设置测量的宽高值，从而完成测量工作。两个 `int` 参数分别为测量的宽度和高度。

通过 `MeasureSpec` 类，我们就获取了 `View` 的测量模式和 `View` 想要绘制的大小。有了这些信息，就可以控制 `View` 最后显示的大小。

### View 绘制

当 View 测量结束后，就可以在 `Canvas` 对象上绘制所需要的图形。 `Canvas` 就是一个画板，使用 `Paint` 可以在上面作画。通常通过继承 View 并重写它的 `onDraw(Canvas)` 方法完成绘图。该方法中有一个参数，就是 `Canvas` 对象。调用 `draw***()` 一系列方法即可绘制相应图形。

在 `onDraw(Canvas)` 方法之外，通常需要使用代码创建一个 `Canvas` 对象。

```java
Canvas canvas = new Canvas(bitmap);
```

当创建一个 `Canvas` 对象时，一般传入一个 `Bitmap` 对象。传入的 `Bitmap` 与通过这个 `Bitmap` 创建的 `Canvas` 对象是紧紧联系在一起的，这个 `Bitmap` 用来存储所有绘制在 `Canvas` 上的像素信息。这个过程称为装载画布。通过这种方式创建的 `Canvas` 对象调用所有的 `draw***()` 方法进行的绘制都发生在这个 `Bitmap` 上。

举例说明。首先在 `onDraw(Canvas)` 方法中绘制两个 `Bitmap` 。

```java
@Override
protected void onDraw(Canvas canvas) {
    canvas.drawBitmap(bitmap1, 0, 0, null);
    canvas.drawBitmap(bitmap2, 0, 0, null);
}
```

将 `bitmap2` 对象装载到另一个 `Canvas` 对象中。

```java
Canvas mCanvas = new Canvas(bitmap2);
```

在 `onDraw(Canvas)` 方法之外使用 `mCanvas.draw***()` 方法在 `bitmap2` 上进行绘图。再刷新 View 时，会发现通过 `onDraw(Canvas)` 方法绘制的 `bitmap2` 已经发生了改变。这是因为 `bitmap2` 承载了在 `mCanvas` 上所进行的绘图操作。虽然并没有将图形直接绘制在 `onDraw(Canvas)` 方法指定的画布上，但是通过改变 `Bitmap` ，然后让 View 进行重绘，依然显示了改变之后的 `Bitmap` 。

### ViewGroup 测量

`ViewGroup` 负责子 View 的显示大小。当 `ViewGroup` 的大小为 `wrap_content` 时， `ViewGroup` 需要对子 View 进行遍历，调用子 View 的 `measure(int, int)` 方法以获得所有子 View 的大小，从而决定自己的大小。而在其他模式下会通过具体的指定值来设置自身的大小。

当子 View 测量完毕后，需要将子 View 放到合适的位置，这个过程是 View 的布局过程。 `ViewGroup` 在执行布局过程时，同样使用遍历来调用子 View 的 `layout(int, int, int, int)` 方法，并指定其具体显示的位置，从而来决定其布局位置。

在自定义 ViewGroup 时，通常会去重写 `onLayout(boolean, int, int, int, int)` 方法来控制其子 View 显示位置的逻辑。同样，如果需要支持 `wrap_content` 属性，那么它还必须重写 `onMeasure(int, int)` 方法。

### ViewGroup 绘制

`ViewGroup` 通常情况下不需要绘制。如果不是指定了 ViewGroup 的背景颜色，那么 `onDraw(Canvas)` 方法都不会被调用。但是， ViewGroup 会使用 `dispatchDraw(Canvas)` 方法绘制其子 View 。过程同样是通过遍历所有子 View ，并调用子 View 的绘制方法来完成绘制工作。

### 自定义 View

在自定义 View 时，通常会重写 `onDraw(Canvas)` 方法绘制 View 的显示内容。如果该 View 需要使用 `wrap_content` 属性，那么还必须重写 `onMeasure(int, int)` 方法。另外，通过自定义 `attrs` 属性，还可以设置新的属性配置值。

在View中通常有以下一些比较重要的回调方法：

- `onFinishInflate()`

    从布局文件 XML 加载组件后回调

- `onSizeChanged()`

    组件大小改变时回调

- `onMeasure()`

    回调该方法来进行测量

- `onLayout()`

    回调该方法来确定显示的位置。

- `onTouchEvent()`

    监听到触摸事件时回调。

#### 拓展现有控件

**例：** 拓展 TextView 创建新的控件。使其背景更加丰富，多绘制几层背景。

原生的 `TextView` 使用 `onDraw(Canvas)` 方法绘制显示的文字。当继承了原生的 `TextView` 之后，如果不重写其 `onDraw(Canvas)` 方法，则不会修改 `TextView` 的任何效果。在自定义的 TextView 中调用 `TextView#onDraw(Canvas)` 方法绘制显示的文字。在调用 `super.onDraw(Canvas)` 方法之前和之后，都可以实现自己的逻辑，分别在系统绘制文字前后，完成自己的操作。

在构造方法中完成必要对象的初始化工作。

```java
// 完成必要对象的初始化工作，如初始化画笔等
mPaint1 = new Paint();
mPaint1.setColor(getResources().getColor(android.R.color.holo_blue_light));
mPaint1.setStyle(Paint.Style.FILL);
mPaint2 = new Paint();
mPaint2.setColor(Color.YELLOW);
mPaint2.setStyle(Paint.Style.FILL);
```

在调用 `super.onDraw(Canvas)` 方法前，也就是在绘制文字之下，绘制两个不同大小的矩形，形成一个重叠效果。再调用 `super.onDraw(Canvas)` 方法，执行绘制文字的工作。

```java
@Override
protected void onDraw(Canvas canvas) {
    // 在回调父类方法前，实现自己的逻辑，对 TextView 来说是在绘制文本内容前
    // 绘制外层矩形
    canvas.drawRect(0, 0, getMeasuredWidth(), getMeasuredHeight(), mPaint1);
    // 绘制内层矩形
    canvas.drawRect(10, 10, getMeasuredWidth() - 10,
            getMeasuredHeight() - 10, mPaint2);
    canvas.save();
    // 绘制文字前向右平移 10 像素
    canvas.translate(10, 0);

    // 父类完成的方法，即绘制文本
    super.onDraw(canvas);

    // 在回调父类方法后，实现自己的逻辑，对 TextView 来说是在绘制文本内容后
    canvas.restore();
}
```

**例：** 拓展 TextView 创建新的控件。利用 `LinearGradient Shader` 和 `Matrix` 实现动态文字闪动效果。

在 `onSizeChanged(int, int, int, int)` 方法中进行对象的初始化工作。使用 `getPaint()` 方法获取当前绘制 TextView 的 `Paint` 对象。根据 `View` 的宽度设置一个 `LinearGradient` 渐变渲染器，并设置给 `Paint` 对象。最后，在 `onDraw(Canvas)` 方法中，通过矩阵的方式来不断平移渐变效果，从而在绘制文字时，产生动态的闪动效果。

```java
@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
    if (mViewWidth == 0) {
        mViewWidth = getMeasuredWidth();
        if (mViewWidth > 0) {
            mPaint = getPaint();
            mLinearGradient = new LinearGradient(0, 0, mViewWidth, 0,
                    new int[]{Color.BLUE, Color.WHITE, Color.BLUE}, null,
                    Shader.TileMode.CLAMP);
            mPaint.setShader(mLinearGradient);
            mGradientMatrix = new Matrix();
        }
    }
}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    if (mGradientMatrix != null) {
        mTranslate += mViewWidth / 30;
        if (mTranslate > 2 * mViewWidth) {
            mTranslate = -mViewWidth;
        }
        mGradientMatrix.setTranslate(mTranslate, 0);
        mLinearGradient.setLocalMatrix(mGradientMatrix);
        postInvalidateDelayed(1000 / 60);
    }
}
```

#### 创建组合控件

创建复合控件可以创建出具有重用功能的控件集合。这种方式通常需要继承一个合适的 `ViewGroup` ，再给它添加指定功能的控件，从而组合成新的复合控件。通过这种方式创建的控件，我们一般会给它指定一些可配置的属性，让它具有更强的拓展性。

对于应用程序中共通的 UI 界面，可以将界面抽象出来，形成一个共通的 UI 组件，在需要添加处引用。还可以给组件增加相应的接口，让调用者能够更加灵活地控制。这样不仅可以提高界面的复用率，更能在需要修改 UI 时，做到快速修改，而不需要对每个页面都进行修改。对于 UI 模板，应该具有 `通用性` 与 `可定制性` 。也就是说，我们需要给调用者以丰富的接口，让他们可以更改模板中的文字、颜色、行为等信息，而不是所有的模板都一样，那样就失去了模板的意义。

**例：** 自定义组合 TopBar 。

- 定义属性

    在工程 `res` 目录的 `values` 目录下创建 `attrs.xml` 属性定义文件，并在该文件中定义相应的属性。通过 `<declare-styleable>` 标签声明使用自定义属性，并通过 `name` 属性确定引用的名称。最后，通过 `<attr>` 标签声明具体的自定义属性，并通过 `format` 属性来指定属性的类型。有些属性可以是多种属性类型，使用 `|` 来分隔不同的属性。

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <declare-styleable name="TopBar">
            <attr name="title" format="string"/>
            <attr name="titleTextColor" format="color"/>
            <attr name="titleTextSize" format="dimension"/>
            <attr name="leftText" format="string"/>
            <attr name="leftTextColor" format="color"/>
            <attr name="leftBackground" format="reference|color"/>
            <attr name="rightText" format="string"/>
            <attr name="rightTextColor" format="color"/>
            <attr name="rightBackground" format="reference|color"/>
        </declare-styleable>
    </resources>
    ```

    创建自定义控件 `TopBar` ，并让它继承自 `RelativeLayout` ，从而组合一些需要的控件。在构造方法使用 `TypedArray#get***()` 获取 XML 布局文件中自定义的属性。当获取完所有的属性值后，需要调用 `TypedArray#recyle()`方法完成资源的回收。

    ```java
    // 将 attrs.xml 中定义的 declare-styleable 的所有属性的值存储到 TypedArray 中
    TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.TopBar);

    // 从 TypedArray 中取出对应的值来为要设置的属性赋值
    mTitle = ta.getString(R.styleable.TopBar_title);
    mTitleTextColor = ta.getColor(R.styleable.TopBar_titleTextColor, 0);
    mTitleTextSize = ta.getDimension(R.styleable.TopBar_titleTextSize, 10);
    mLeftText = ta.getString(R.styleable.TopBar_leftText);
    mLeftTextColor = ta.getColor(R.styleable.TopBar_leftTextColor, 0);
    mLeftBackground = ta.getDrawable(R.styleable.TopBar_leftBackground);
    mRightText = ta.getString(R.styleable.TopBar_rightText);
    mRightTextColor = ta.getColor(R.styleable.TopBar_rightTextColor, 0);
    mRightBackground = ta.getDrawable(R.styleable.TopBar_rightBackground);

    // 获取完 TypedArray 的值后，要调用 recycle() 方法来避免重新创建的时候的错误
    ta.recycle();
    ```

- 组合控件

    UI 模板 `TopBar` 由三个控件组成。左边的点击按钮 `mLeftButton` ，右边的点击按钮 `mRightButton` 和中间的标题栏 `mTitleView` 。

  - 添加组件

    通过动态添加控件的方式，使用 `addView()` 方法将这三个控件加入到定义的 `TopBar` 模板中，并给它们设置我们前面所获取到的具体的属性值。

    ```java
    // 创建组件
    mLeftButton = new Button(context);
    mRightButton = new Button(context);
    mTitleView = new TextView(context);

    // 为创建的组件元素赋值，值来源于我们在引用的 xml 文件中给对应属性的赋值
    mTitleView.setText(mTitle);
    mTitleView.setTextColor(mTitleTextColor);
    mTitleView.setTextSize(mTitleTextSize);
    mTitleView.setGravity(Gravity.CENTER);
    mLeftButton.setText(mLeftText);
    mLeftButton.setTextColor(mLeftTextColor);
    mLeftButton.setBackground(mLeftBackground);
    mRightButton.setText(mRightText);
    mRightButton.setTextColor(mRightTextColor);
    mRightButton.setBackground(mRightBackground);

    // 为组件元素设置相应的布局元素并添加到 ViewGroup
    mTitleParams = new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.MATCH_PARENT);
    mTitleParams.addRule(RelativeLayout.CENTER_IN_PARENT, TRUE);
    addView(mTitleView, mTitleParams);

    mLeftParams = new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.MATCH_PARENT);
    mLeftParams.addRule(RelativeLayout.ALIGN_PARENT_LEFT, TRUE);
    addView(mLeftButton, mLeftParams);

    mRightParams = new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.MATCH_PARENT);
    mRightParams.addRule(RelativeLayout.ALIGN_PARENT_RIGHT, TRUE);
    addView(mRightButton, mRightParams);
    ```

  - 定义接口

    为按钮添加点击事件。通过接口回调的思想，将点击事件的具体实现逻辑交给调用者添加。

    ```java
    // 接口对象，实现回调机制。在回调方法中通过映射的接口对象调用接口中的方法而不用去考虑如何实现
    public interface TopBarClickListener {
        //左按钮点击事件
        void leftClick();

        //右按钮点击事件
        void rightClick();
    }
    ```

  - 暴露接口

    在模板方法中为左、右按钮增加点击事件，但不去实现具体的逻辑，而是调用接口中相应的点击方法。

    ```java
    //暴露一个方法给调用者注册接口回调。通过接口获得回调者对接口方法的实现
    public void setOnTopBarClickListener(TopBarClickListener listener) {
        mListener = listener;
    }
    ```

    ```java
    //按钮的点击事件，不需要具体的实现，只需调用接口的方法，回调的时候，会有具体的实现
    mLeftButton.setOnClickListener(v -> {
        if (null != mListener) {
            mListener.leftClick();
        }
    });

    mRightButton.setOnClickListener(v -> {
        if (null != mListener) {
            mListener.rightClick();
        }
    });
    ```

  - 实现回调

    在调用代码中，需要实现个接口，并完成接口中的方法，确定具体的实现逻辑。然后使用暴露的方法，将接口的对象传入，从而完成回调。

    ```java
    mTopBar.setOnTopBarClickListener(
            new TopBarClickListener() {
                @Override
                public void leftClick() {
                    // 点击左侧按钮的逻辑
                }

                @Override
                public void rightClick() {
                    // 点击右侧按钮的逻辑
                }
            });
    ```

  - 动态修改 UI

    可以使用公共方法动态修改 UI 模板中的 UI ，进一步提高模板的可定制性。

    ```java
    public static final int BUTTON_LEFT = 1;
    public static final int BUTTON_RIGHT = 2;

    /**
     * 设置按钮的显示与否通过 id 区分按钮， flag 区分是否显示
     *
     * @param id   id
     * @param flag 是否显示
     */
    public void setButtonVisibility(int id, boolean flag) {
        switch (id) {
            case BUTTON_LEFT:
                mLeftButton.setVisibility(flag ? VISIBLE : GONE);
                break;

            case BUTTON_RIGHT:
                mRightButton.setVisibility(flag ? VISIBLE : GONE);
                break;
        }
    }
    ```

    当调用 `TopBar` 对象中该方法后，根据参数就可以动态地控制按钮的显示。

    ```java
    // 控制 TopBar 上组件的状态
    mTopbar.setButtonVisibility(BUTTON_LEFT, true);
    mTopbar.setButtonVisibility(BUTTON_RIGHT, false);
    ```

- 引用 UI 模版

    在需要使用的地方引用 UI 模板。在引用前，需要使用 `xmlns` 属性指定引用第三方控件的命名空间。

    ```xml
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:custom="http://schemas.android.com/apk/res-auto"
    ```

    将引入的第三方控件的命名空间取名为 `custom` ，之后在 XML 文件中使用自定义的属性时，就可以通过这个命名空间引用。使用自定义 View 时，需要指定完整的包名。

    ```xml
    <com.jpfeng.test.widget.TopBar
        android:id="@+id/topBar"
        android:layout_width="match_parent"
        android:layout_height="56dp"
        custom:leftBackground="@drawable/blue_button"
        custom:leftText="@String/text_button_left"
        custom:leftTextColor="@color/text_button"
        custom:rightBackground="@drawable/blue_button"
        custom:rightText="@String/text_button_right"
        custom:rightTextColor="@color/text_button"
        custom:title="@String/text_button_title"
        custom:titleTextColor="@color/text_title"
        custom:titleTextSize="16sp"/>
    ```

    如果将 UI 模板写到布局文件中，就可以在其他布局文件中，直接引用这个 UI 模板。

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <com.jpfeng.test.widget.TopBar
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:custom="http://schemas.android.com/apk/res-auto"
        android:id="@+id/topBar"
        android:layout_width="match_parent"
        android:layout_height="56dp"
        custom:leftBackground="@drawable/blue_button"
        custom:leftText="@String/text_button_left"
        custom:leftTextColor="@color/text_button"
        custom:rightBackground="@drawable/blue_button"
        custom:rightText="@String/text_button_right"
        custom:rightTextColor="@color/text_button"
        custom:title="@String/text_button_title"
        custom:titleTextColor="@color/text_title"
        custom:titleTextSize="16sp"/>
    ```

    在其他的布局文件中，直接通过 `<include>` 标签引用 UI 模板。

    ```xml
    <include layout="@layout/topbar"/>
    ```

#### 重写 View 实现全新控件

创建一个自定义 View ，难点在于绘制控件和实现交互。通常需要继承 `View` 类，并重写 `onDraw(Canvas)` `onMeasure(int, int)` 等方法实现绘制逻辑，同时通过重写 `onTouchEvent(MotionEvent)` 等触控事件来实现交互逻辑。还可以通过引入自定义属性，丰富自定义 View 的可定制性。

**例：** 弧线展示图

这个自定义 View 分为三个部分，分别是中间的圆形、中间显示的文字和外圈的弧线。在 `onDraw(Canvas)` 方法中一个个绘制即可。图形如果单独绘制，是非常容易的。这里进行了一下
组合，就创建了一个新的 View 。

在初始化时，设置好绘制三种图形的参数。圆和弧线的代码如下所示。绘制文字，只需要设置好文字的起始绘制位置。

```java
// 圆
mCircleXY = length / 2;
mRadius = (float) (length * 0.5 / 2);

// 弧线
mArcRectF = new RectF(
(float) (length * 0.1),
(float) (length * 0.1),
(float) (length * 0.9),
(float) (length * 0.9));
```

在 `onDraw(Canvas)` 方法中进行绘制。

```java
//绘制圆
canvas.drawCircle(mCircleXY, mCircleXY, mRadius, mCirclePaint);
//绘制弧线
canvas.drawArc(mArcRectF, 270, mSweepAngle, false, mArcPaint);
//绘制文字
canvas.drawText(mShowText, 0, mShowText.length(),
        mCircleXY, mCircleXY + (mShowTextSize / 4), mTextPaint);
```

创建一些方法让调用者设置不同的状态值。当用户不指定具体的比例值时，默认设置为 25 。

```java
public void setSweepValue(float sweepValue) {
    if (sweepValue != 0) {
        mSweepValue = sweepValue;
    } else {
        mSweepValue = 25;
    }

    this.invalidate();
}
```

**例：** 音频条形图

实现类似 PC 上某些音乐播放器根据音频音量大小显示的音频条形图。由于只是演示自定义 View 的用法，我们不去真实地监听音频输入，随机模拟一些数字。

绘制一个个矩形，每个矩形之间稍微偏移一点距离。通过 `Math.random()` 方法随机改变这些高度值，并赋值给 `currentHeight` 。最后，使用 `postInvalidateDelayed(int)` 方法进行 View 的延迟重绘。

```java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    for (int i = 0; i < mRectCount; i++) {
        mRandom = Math.random();
        float currentHeight = (float) (mRectHeight * mRandom);
        canvas.drawRect((float) (mWidth * 0.4 / 2 + mRectWidth * i + offset), currentHeight,
                (float) (mWidth * 0.4 / 2 + mRectWidth * (i + 1)), mRectHeight, mPaint);
    }

    postInvalidateDelayed(300);
}
```

为了使自定义 View 更加逼真，可以在绘制小矩形的时候，给 `Paint` 对象增加一个 `LinearGradient` 渐变效果。

```java
@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
    mWidth = getWidth();
    mRectHeight = getHeight();
    mRectWidth = (int) (mWidth * 0.6 / mRectCount);
    mLinearGradient = new LinearGradient(0, 0, mRectWidth, mRectHeight,
            Color.YELLOW, Color.BLUE, Shader.TileMode.CLAMP);
    mPaint.setShader(mLinearGradient);
}
```

### 自定义 ViewGroup

自定义 ViewGroup 通常需要重写 `onMeasure(int, int)` 方法来对子 View 进行测量。重写 `onLayout(boolean, int, int, int, int)` 方法来确定子 View 的位置。重写 `onTouchEvent(MotionEvent)` 方法增加响应事件。

**例：** 实现 `ScrollView` 所具有的上下滑动功能。但是在滑动的过程中，增加一个黏性的效果。即当一个子 View 向上滑动大于一定的距离后，松开手指，它将自动向上滑动，显示下一个子 View 。

首先放置好子 View 。使用遍历的方式通知子 View 对自身进行测量。

> 控件正确的高度应为 `screenHeight * childCount` ，即子 View 的个数乘以屏幕的高度。书中使用设置 `LayoutParams` 的方式改变 ViewGroup 的高度。经实践，这样设置后调用 `getHeight()` 方法总是返回控件在屏幕占有的高度，无法返回正确的控件高度。在此调用 `setMeasuredDimension(int, int)` 方法控制控件的测量高度，从而使 `getHeight()` 方法返回正确的控件高度。

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);

    int count = getChildCount();
    for (int i = 0; i < count; ++i) {
        View childView = getChildAt(i);
        measureChild(childView, widthMeasureSpec, heightMeasureSpec);
    }

    int h = MeasureSpec.makeMeasureSpec(mScreenHeight * count,
            MeasureSpec.getMode(heightMeasureSpec));
    setMeasuredDimension(widthMeasureSpec, h);
}
```

通过遍历来设定子 View 放置的位置。直接调用子 View 的 `layout(int, int, int, int)` 方法，并将具体的位置作为参数传入。主要修改每个子 View 的 `top` 和 `bottom` 属性，让它们依次排列。

```java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    int childCount = getChildCount();

    // 设置ViewGroup的高度
    // MarginLayoutParams mlp = (MarginLayoutParams) getLayoutParams();
    // mlp.height = mScreenHeight * childCount;
    // setLayoutParams(mlp);

    for (int i = 0; i < childCount; i++) {
        View child = getChildAt(i);
        if (child.getVisibility() != View.GONE) {
            child.layout(l, i * mScreenHeight, r, (i + 1) * mScreenHeight);
        }
    }
}
```

重写 `onTouchEvent(MotionEvent)` 方法，为 ViewGroup 添加响应事件。使用 `scrollBy(int, int)` 方法辅助滑动。在 `MotionEvent` 的 `ACTION_MOVE` 事件中，调用 `scrollBy(0, dy)` 方法，手指滑动时让 ViewGroup 中的所有子 View 也跟着滚动 dy 距离即可。在 `ACTION_UP` 事件中判断手指滑动的距离。如果超过一定距离，则使用 `Scroller` 类平滑移动到下一个子 View 。如果小于一定距离，则回滚原来的位置。

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    int y = (int) event.getY();
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            mLastY = y;
            mStart = getScrollY();
            break;

        case MotionEvent.ACTION_MOVE:
            if (!mScroller.isFinished()) {
                mScroller.abortAnimation();
            }

            int dy = mLastY - y;
            if (getScrollY() < 0) {
                dy = 0;
            }

            if (getScrollY() > getHeight() - mScreenHeight) {
                dy = 0;
            }

            scrollBy(0, dy);
            mLastY = y;
            break;

        case MotionEvent.ACTION_UP:
            mEnd = getScrollY();
            int dScrollY = mEnd - mStart;

            if (dScrollY > 0) {
                if (dScrollY < mScreenHeight / 3) {
                    mScroller.startScroll(0, getScrollY(), 0, -dScrollY);
                } else {
                    mScroller.startScroll(0, getScrollY(), 0, mScreenHeight - dScrollY);
                }

            } else {
                if (-dScrollY < mScreenHeight / 3) {
                    mScroller.startScroll(0, getScrollY(), 0, -dScrollY);
                } else {
                    mScroller.startScroll(0, getScrollY(), 0, -mScreenHeight - dScrollY);
                }
            }
            break;
    }

    postInvalidate();
    return true;
}
```

最后加上 `computeScroll()` 的代码。

```java
@Override
public void computeScroll() {
    super.computeScroll();
    if (mScroller.computeScrollOffset()) {
        scrollTo(0, mScroller.getCurrY());
        postInvalidate();
    }
}
```

这样就实现了一个简单的黏性 ScrollView 。其中仍然存在一些 bug ，并且还可以进一步添加一些功能，比如滑动的惯性效果等。这些功能可以在后面慢慢添加，这也是一个控件的迭代过程。

### 事件拦截机制

***TODO***

[↑ 目录](#index)

------

<h2 id="chap4">ListView 使用</h2>

***TODO***

[↑ 目录](#index)

------

<h2 id="chap5">Android Scroll 分析</h2>

### 坐标系和触控事件

滑动一个 View 本质就是移动一个 View ，改变其当前所处的位置。它的原理是不断改变 View 的坐标来实现这一效果。实现 View 的滑动，必须监听用户触摸的事件，并根据事件传入的坐标，动态且不断地改变 View 的坐标，从而实现 View 跟随用户触摸的滑动而滑动。

在 Android 中，将屏幕最左上角的顶点作为 Android 坐标系的原点。从这个点向右是 X 轴正方向，从这个点向下是 Y 轴正方向。系统提供了 `getLocationOnScreen(int[])` 方法获取 Android 坐标系中点的位置，即该视图左上角在 Android 坐标系中的坐标。在触控事件中使用 `getRawX()` 和 `getRawY()` 方法获得的坐标同样是 Android 坐标系中的坐标。

除此之外还有视图坐标系，它描述子视图在父视图中的位置关系。视图坐标系同样是以原点向右为 X 轴正方向，以原点向下为 Y 轴正方向。但是在视图坐标系中，原点不再是 Android 坐标系中的屏幕左上角，而是以父视图左上角为坐标原点。在触控事件中通过 `getX()` 和 `getY()` 获得的坐标是视图坐标系中的坐标。

```text
坐标原点 (0,0)
    .-----------→ x轴
    |
    |
    |
    |
    |
    ↓ y轴
```

触控事件 `MotionEvent` 在用户交互中，占着举足轻重的地位。 `MotionEvent` 中封装的一些常用的事件常量，定义了触控事件的不同类型。

```java
// 单点触摸按下动作
public static final int ACTION_DOWN = 0;
// 单点触摸离开动作
public static final int ACTION_UP = 1;
// 触摸点移动动作
public static final int ACTION_MOVE = 2;
// 触摸动作取消
public static final int ACTION_CANCEL = 3;
// 触摸动作超出边界
public static final int ACTION_OUTSIDE = 4;
// 多点触摸按下动作
public static final int ACTION_POINTER_DOWN = 5;
// 多点离开动作
public static final int ACTION_POINTER_UP = 6;
```

通常，在 `onTouchEvent(MotionEvent)` 方法中通过 `event.getAction()` 方法获取触控事件的类型，并使用 `switch-case` 进行筛选。代码的模式基本固定。在不涉及多点操作的情况下，通常可以使用以下代码完成触控事件的监听。

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    //获取当前输入点的X、Y坐标（视图坐标）
    int x = (int) event.getX();
    int y = (int) event.getY();

    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
        //处理输入的按下事件
        break;

        case MotionEvent.ACTION_MOVE:
        //处理输入的移动事件
        break;

        case MotionEvent.ACTION_UP:
        //处理输入的离开事件
        break;
    }
    return true;
}
```

在 Android 中，系统提供了非常多的方法来获取坐标值、相对距离等。

- `View` 提供的获取坐标方法

  - `getTop()`

    获取到的是 View 自身的顶边到其父布局顶边的距离

  - `getLeft()`

    获取到的是 View 自身的左边到其父布局左边的距离

  - `getRight()`

    获取到的是 View 自身的右边到其父布局左边的距离

  - `getBottom()`

    获取到的是 View 自身的底边到其父布局顶边的距离

- `MotionEvent` 提供的获取坐标方法

  - `getX()`

    获取点击事件距离控件左边的距离，即视图坐标

  - `getY()`

    获取点击事件距离控件顶边的距离，即视图坐标

  - `getRawX()`

    获取点击事件距离整个屏幕左边的距离，即绝对坐标

  - `getRawY()`

    获取点击事件距离整个屏幕顶边的距离，即绝对坐标

### 滑动效果的实现

实现滑动效果的思想基本是一致的。当触摸 View 时，系统记下当前触摸点坐标。当手指移动时，系统记下移动后的触摸点坐标，从而获取到相对于前一次坐标点的偏移量，并通过偏移量来修改
View的坐标。不断重复，从而实现滑动过程。

- `layout(int, int, int, int)`

    可以通过修改 View 的 `left` ， `top` ， `right` ， `bottom` 四个属性来控制 View 的坐标。在 `ACTION_DOWN` 事件中记录触摸点的坐标，在 `ACTION_MOVE` 事件中计算偏移量。将偏移量作用到 `layout(int, int, int, int)` 方法中，在目前 Layout 的 `left` ， `top` ， `right` ， `bottom` 基础上，增加计算出来的偏移量。每次移动后， View 都会调用 `layout(int, int, int, int)` 方法重新布局，从而达到移动 View 的效果。

    可使用 `getX()` 和 `getY()` 方法获取坐标值，即通过视图坐标来获取偏移量。

    ```java
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 记录触摸点坐标
                lastX = x;
                lastY = y;
            break;

            case MotionEvent.ACTION_MOVE:
                // 计算偏移量
                int offsetX = x - lastX;
                int offsetY = y - lastY;

                // 在当前 left ， top ， right ， bottom 的基础上加上偏移量
                layout(getLeft() + offsetX, getTop() + offsetY,
                        getRight() + offsetX, getBottom() + offsetY);
            break;
        }

        return true;
    }
    ```

    还可以使用 `getRawX()` 和 `getRawY()` 获取坐标，并使用绝对坐标来计算偏移量。使用绝对坐标系，在每次执行 `ACTION_MOVE` 的逻辑后，要重新设置初始坐标才能准确地获取偏移量。

    ```java
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int rawX = (int) (event.getRawX());
        int rawY = (int) (event.getRawY());

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 记录触摸点坐标
                lastX = rawX;
                lastY = rawY;
            break;

            case MotionEvent.ACTION_MOVE:
                // 计算偏移量
                int offsetX = rawX - lastX;
                int offsetY = rawY - lastY;

                // 在当前 left ， top ， right ， bottom 的基础上加上偏移量
                layout(getLeft() + offsetX, getTop() + offsetY,
                        getRight() + offsetX, getBottom() + offsetY);

                // 重新设置初始坐标
                lastX = rawX;
                lastY = rawY;
            break;
        }

        return true;
    }
    ```

- `offsetLeftAndRight(int)` 与 `offsetTopAndBottom(int)`

    该方法相当于系统提供的对左右、上下移动的 API 封装。当计算出偏移量后，只需要使用如下代码就可以完成 View 的重新布局。效果与使用 `layout(int, int, int, int)` 方法相同。计算 `offset` 的方法与使用 `layout(int, int, int, int)` 方法时相同。

    ```java
    // 同时对 left 和 right 进行偏移
    offsetLeftAndRight(offsetX);

    // 同时对 top 和 bottom 进行偏移
    offsetTopAndBottom(offsetY);
    ```

- `LayoutParams`

    `LayoutParams` 保存 View 的布局参数。可以在程序中，通过改变 `LayoutParams` 来动态修改一个布局的位置参数，从而达到改变 View 位置的效果。可以使用 `getLayoutParams()` 方法获取 View 的 `LayoutParams` ，并且需要根据 View 所在父布局的类型来转换不同的类型。偏移量的方法与使用 `layout(int, int, int, int)` 方法中计算 offset 一样。在通过改变 `LayoutParams` 来改变一个 View 的位置时，通常改变的是这个 View 的 `Margin` 属性。当获取到偏移量之后，可以通过`setLayoutParams(LayoutParams)` 方法改变其 `LayoutParams` 。

    ```java
    LinearLayout.LayoutParams layoutParams = (LinearLayout.LayoutParams) getLayoutParams();
    layoutParams.leftMargin = getLeft() + offsetX;
    layoutParams.topMargin = getTop() + offsetY;
    setLayoutParams(layoutParams);
    ```

    还可以使用 `ViewGroup.MarginLayoutParams` 实现这样的功能。这样更加方便，不需要考虑父布局的类型。

    ```java
    ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();
    layoutParams.leftMargin = getLeft() + offsetX;
    layoutParams.topMargin = getTop() + offsetY;
    setLayoutParams(layoutParams);
    ```

- `scrollTo(int, int)` 与 `scrollBy(int, int)`

    在 View 中提供了 `scrollTo(int, int)` 和 `scrollBy(int, int)` 两种方式改变一个 View 的位置。这两个方法的区别为， `scrollTo(x, y)` 表示移动到一个具体坐标点 (x, y) ，而 `scrollBy(dx, dy)` 表示移动的增量为 dx 和 dy 。如果在 `ViewGroup` 中使用 `scrollTo(int, int)` 或 `scrollBy(int, int)` 方法，移动的将是所有子 View 。如果在 View 中使用，移动的将是 View 的内容。在 View 中使用这两个方法拖动这个 View ，应该在 View 所在的 ViewGroup 中使用来移动它的子 View 。如果将参数 `dx` 和 `dy` 设置为正数，那么 content 将向坐标轴负方向移动。如果将参数 `dx` 和 `dy` 设置为负数，那么 content 将向坐标轴正方向移动。如果使用 `layout(int, int, int, int)` 时计算偏移量的方法，要实现跟随手指移动而滑动的效果，就必须将偏移量改为负值。在使用绝对坐标时，也可以通过使用 `scrollTo(x, y)` 方法实现这一效果。

    ```java
    int offsetX = x - lastX;
    int offsetY = y - lastY;
    ((View) getParent()).scrollBy(-offsetX, -offsetY);
    ```

- `Scroller`

    使用 `scrollTo(int, int)` 或 `scrollBy(int, int)` 方法，子 View 的平移都是瞬间发生的，在事件执行的时候平移就已经完成了。而 `Scroller` 类可以实现平滑移动的效果，而不是瞬间完成的移动。

    **例：** 让子 View 跟随手指滑动。但是在手指离开屏幕时，让子 View 平滑的移动到初始位置。

  - 初始化 `Scroller`

    通过它的构造方法来创建一个 `Scroller` 对象。

    ```java
    // 初始化 Scroller
    mScroller = new Scroller(context);
    ```

  - 重写 `computeScroll()` 方法，实现模拟滑动

    系统会在绘制 View 时在 `draw(Canvas)` 中调用该方法。该方法实际上使用 `scrollTo(int, int)` 方法，结合 `Scroller` 对象，帮助获取当前的滚动值。通过不断地瞬间移动一个小的距离，实现整体上的平滑移动效果。 `Scroller` 类提供了 `computeScrollOffset()` 方法判断是否完成了整个滑动。同时也提供了 `getCurrX()` 和 `getCurrY()` 方法来获得当前的滑动坐标。在代码中需要注意 `invalidate()` 方法。因为只能在 `computeScroll()` 方法中获取模拟过程中的 `scrollX` 和 `scrollY` 坐标。但 `computeScroll()` 方法不会自动调用，只能通过 `invalidate() → draw(Canvas) → computeScroll()` 来间接调用 `computeScroll()` 方法。所以需要在代码中调用 `invalidate()` 方法，实现循环获取 `scrollX` 和 `scrollY` 的目的。当模拟过程结束后， `Scroller#computeScrollOffset()` 方法会返回 `false` ，从而中断循环，完成整个平滑移动过程。

    ```java
    @Override
    public void computeScroll() {
        super.computeScroll();

        // 判断Scroller是否执行完毕
        if (mScroller.computeScrollOffset()) {
            ((View) getParent()).scrollTo(mScroller.getCurrX(), mScroller.getCurrY());

            // 通过重绘来不断调用 computeScroll
            invalidate();
        }
    }
    ```

  - `startScroll()` 开启模拟过程

    `startScroll()` 方法具有两个重载方法。它们的区别就是一个具有指定的持续时长，而另一个没有。其他四个坐标，与它们的命名含义相同，是起始坐标与偏移量。

    - public void startScroll(int startX, int startY, int dx, int dy, int duration)
    - public void startScroll(int startX, int startY, int dx, int dy)

    可以使用 `getScrollX()` 和 `getScrollY()` 方法获取父视图中 content 所滑动到的点的坐标。要监听手指离开屏幕的事件，需要在 `onTouchEvent(NotionEvent)` 中增加一个 `ACTION_UP` 监听选项。并在该事件中通过调用 `startScroll()` 方法完成平滑移动。需要注意的还是 `invalidate()` 方法，要使用这个方法来通知 View 进行重绘，从而调用 `computeScroll()` 的模拟过程。也可以给 `startScroll()` 方法增加一个 `duration` 参数设置滑动的持续时长。

    ```java
    case MotionEvent.ACTION_UP:
        //手指离开时，执行滑动过程
        View viewGroup = ((View) getParent());
        mScroller.startScroll(viewGroup.getScrollX(), viewGroup.getScrollY(),
                -viewGroup.getScrollX(), -viewGroup.getScrollY());
        invalidate();
        break;
    ```

- 属性动画

    可以使用属性动画完成 View 的滑动特效。在 [第七章](#chap7) 中将详细讲解如何使用属性动画控制 View 的移动效果。

- `ViewDragHelper`

    通过 `ViewDragHelper` 类基本可以实现各种不同的滑动、拖放需求。

    **例：** 实现类似 QQ 滑动侧边栏的布局。初始时显示内容界面。当用户手指滑动超过一段距离时，内容界面侧滑显示菜单界面。

  - 初始化 `ViewDragHelper`

    `ViewDragHelper` 通常定义在一个 `ViewGroup` 的内部，并通过其静态工厂方法进行初始化。第一个参数是要监听的 `View` ，通常是一个 `ViewGroup` ，即 `parentView` 。第二个参数是 `Callback` 回调，这个回调是整个 `ViewDragHelper` 的逻辑核心。

    ```java
    mViewDragHelper = ViewDragHelper.create(this, callback);
    ```

  - 拦截事件

    重写事件拦截方法，将事件传递给 `ViewDragHelper` 进行处理。

    ```java
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return mViewDragHelper.shouldInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        // 将触摸事件传递给 ViewDragHelper ，此操作必不可少
        mViewDragHelper.processTouchEvent(event);
        return true;
    }
    ```

  - 处理 `computeScroll()`

    使用 `ViewDragHelper` 需要重写 `computeScroll()` 方法。因为 `ViewDragHelper` 内部是通过 `Scroller` 实现平滑移动的。

    ```java
    @Override
    public void computeScroll() {
        if (mViewDragHelper.continueSettling(true)) {
            ViewCompat.postInvalidateOnAnimation(this);
        }
    }
    ```

  - 处理回调 `ViewDragHelper.Callback`

    创建 `ViewDragHelper.Callback` 对象。需要重写 `tryCaptureView(View, int)` 方法。通过这个方法，可以指定在创建 `ViewDragHelper` 时，参数 `parentView` 中的哪一个子 View 可以被移动。这里指定 `MainView` 是可以被拖动的。

    ```java
    @Override
    public boolean tryCaptureView(View child, int pointerId) {
        // 如果当前触摸的 child 是 mMainView 时开始检测
        return mMainView == child;
    }
    ```

    具体的滑动方法有 `clampViewPositionVertical(View, int, int)` 和 `clampViewPositionHorizontal(View, int, int)` ，分别对应垂直和水平方向上的滑动。如果要实现滑动效果，那么必须要重写这两个方法。因为它默认的返回值为 0 ，即不发生滑动。只重写其中的一个，就只会实现该方向上的滑动效果。方法中第二个 `int` 参数代表在方向上子 View 移动的距离。而第三个 `int` 参数表示比较前一次的增量。通常只需要返回 `top` 和 `left` 即可。但当需要更加精确地计算 `padding` 等属性的时候，就需要进行一些处理，并返回合适大小的值。

    ```java
    // 处理垂直滑动
    @Override
    public int clampViewPositionVertical(View child, int top, int dy) {
        return 0;
    }

    // 处理水平滑动
    @Override
    public int clampViewPositionHorizontal(View child, int left, int dx) {
        return left;
    }
    ```

    此时已经可以实现最基本的滑动效果。当用手拖动 `MainView` 的时候，它就可以跟随手指的滑动而滑动了。

    在 `ViewDragHelper.Callback` 中提供了 `onViewReleased(View, float, float)` 方法。通过重写这个方法，可以实现当手指离开屏幕后的操作。设置让 `MainView` 移动后左边距小于 500px 时，使用 `smoothSlideViewTo(View, int, int)` 方法将 `MainView` 还原到初始状态，即坐标为 `(0, 0)` 的点。而当其左边距大于 500px 时，将 `MainView` 移动到 `(300, 0)` 坐标，即显示 `MenuView` 。

    ```java
    @Override
    public void onViewReleased(View releasedChild, float xvel, float yvel) {
        super.onViewReleased(releasedChild, xvel, yvel);

        // 手指抬起后缓慢移动到指定位置
        if (mMainView.getLeft() < 500) {
            // 关闭菜单
            // 相当于 Scroller#startScroll() 方法
            mViewDragHelper.smoothSlideViewTo(mMainView, 0, 0);
            ViewCompat.postInvalidateOnAnimation(MenuLayout.this);

        } else {
            // 打开菜单
            mViewDragHelper.smoothSlideViewTo(mMainView, 300, 0);
            ViewCompat.postInvalidateOnAnimation(MenuLayout.this);
        }
    }
    ```

    在自定义 ViewGroup 的 `onFinishInflate()` 方法中，按顺序将子 View 分别定义成 `MenuView` 和 `MainView` ，并在 `onSizeChanged(int, int, int, int)` 方法中获得 View 的宽度。如果需要根据 View 的宽度处理滑动后的效果，可以使用这个值进行判断。

    ```java
    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        mMenuView = getChildAt(0);
        mMainView = getChildAt(1);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mWidth = mMenuView.getMeasuredWidth();
    }
    ```

    在 `ViewDragHelper.Callback` 中，系统定义了大量的监听方法处理各种事件。例如：

    - `onViewCaptured(View, int)`

        在用户触摸到 View 后回调。

    - `onViewDragStateChanged(int)`

        在拖拽状态改变时回调，如 `idle` ， `dragging` 等状态。

    - `onViewPositionChanged(View, int, int, int, int)`

        在位置改变时回调，常用于滑动时更改 `scale` 进行缩放等效果。

[↑ 目录](#index)

------

<h2 id="chap6">Android 绘图机制</h2>

### 屏幕信息

- 屏幕大小

    指屏幕对角线的长度。通常使用 `英寸` 来度量。

- 分辨率

    指手机屏幕的像素点个数。例如 `1080 × 1920` 就是指屏幕的分辨率。指宽有 `1080` 个像素点，而高有 `1920` 个像素点。

- PPI

    每英寸像素 (`Pixels Per Inch`) 又被称为 `DPI` (`Dots Per Inch`) 。它是由对角线的像素点数除以屏幕的大小得到的。

- 屏幕密度

    每个厂商的 Android 手机具有不同的大小尺寸和像素密度的屏幕。 Android 系统如果要精确到每种 DPI 的屏幕，那基本是不可能的。因此，系统定义了几个标准的 DPI 值，作为手机的固定 DPI 。

    | 密度    | 密度值 | 分辨率       |
    | :----: | :---: | :---------: |
    | ldpi   | 120   | 240 × 320   |
    | mdpi   | 160   | 320 × 480   |
    | hdpi   | 240   | 480 × 800   |
    | xhdpi  | 320   | 720 × 1280  |
    | xxhdpi | 480   | 1080 × 1920 |

- 独立像素密度

    由于屏幕密度的不同，导致同样像素大小的长度，在不同密度的屏幕上显示长度不同。因为相同长度的屏幕，高密度的屏幕包含更多的像素点。 Android 系统使用 `mdpi` 屏幕作为标准，在这个屏幕上 `1px = 1dp` 。其他屏幕则可以通过比例进行换算。我们可以得出在各个密度值中的换算公式，在 `mdpi` 中 `1dp = 1px` ，在 `hdpi` 中 `1dp = 1.5px` ，在 `xhdpi` 中 `1dp = 2px` ，在 `xxhdpi` 中 `1dp = 3px` 。

#### 单位转换

在程序中，可以非常方便地对这些单位进行转换。

```java
/**
 * dp sp px 转换工具类
 */
public class DisplayUtil {

    /**
     * 将 px 值转换为 dp 值，保证尺寸大小不变
     *
     * @param pxValue px 值
     * @return dp 值
     */
    public static int px2dp(Context context, float pxValue) {
        // 换算比例， DisplayMetrics 类中属性 density
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (pxValue / scale + 0.5f);
    }

    /**
     * 将 dp 值转换为 px 值，保证尺寸大小不变
     *
     * @param dipValue dp 值
     * @return px 值
     */
    public static int dp2px(Context context, float dipValue) {
        // 换算比例， DisplayMetrics 类中属性 density
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (dipValue * scale + 0.5f);
    }

    /**
     * 将 dp 值转换为 px 值，使用 TypedValue 类进行转换
     *
     * @param dp dp 值
     * @return px 值
     */
    public static int dp2px(Context context, int dp) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dp,
                context.getResources().getDisplayMetrics());
    }

    /**
     * 将 px 值转换为 sp 值，保证文字大小不变
     *
     * @param pxValue px 值
     * @return sp 值
     */
    public static int px2sp(Context context, float pxValue) {
        // 换算比例， DisplayMetrics 类中属性 scaledDensity
        final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int) (pxValue / fontScale + 0.5f);
    }

    /**
     * 将 sp 值转换为 px 值，保证文字大小不变
     *
     * @param spValue sp 值
     * @return px 值
     */
    public static int sp2px(Context context, float spValue) {
        // 换算比例， DisplayMetrics 类中属性 scaledDensity
        final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int) (spValue * fontScale + 0.5f);
    }

    /**
     * 将 sp 值转换为 px 值，使用 TypedValue 类进行转换
     *
     * @param sp sp 值
     * @return px 值
     */
    public static int sp2px(Context context, int sp) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP, sp,
                context.getResources().getDisplayMetrics());
    }
}
```

### 2D 绘图

系统的 `Canvas` 对象提供了各种绘制图像的 API 。 `Paint` 功能也很强大，列举一些它的属性和对应的功能：

- `setAntiAlias(boolean)`

    设置画笔的抗锯齿效果

- `setColor(int)`

    设置画笔的颜色

- `setARGB(int, int, int, int)`

    设置画笔的 `A` `R` `G` `B` 值

- `setAlpha(int)`

    设置画笔的 `Alpha` 值

- `setTextSize(float)`

    设置字体的尺寸

- `setStyle(Style)`

    设置画笔的风格，如空心或实心等

    ```java
    // 空心风格
    paint.setStyle(Paint.Style.STROKE);
    // 实心风格
    paint.setStyle(Paint.Style.FILL);
    ```

- `setStrokeWidth(float)`

    设置空心边框的宽度

`Canvas` 对象提供的绘制图像 API ：

- `drawPoint(float, float, Paint)`

    绘制点。

    ```java
    canvas.drawPoint(x, y, paint);
    ```

- `drawLine(float, float, float, float, Paint)`

    绘制直线。

    ```java
    canvas.drawLine(startX, startY, endX, endY, paint);
    ```

- `drawLines(float[], Paint)`

    绘制多条直线。

    ```java
    float[] pts = {
            startX1, startX1, endX1, endY1,
            ...
            startXn, startYn, endYn, endYn};
    canvas.drawLines(pts, paint);
    ```

- `drawRect(float, float, float, float, Paint)`

    绘制矩形。

    ```java
    canvas.drawRect(left, top, right, bottom, paint);
    ```

- `drawRoundRect(float, float, float, float, float, float, Paint)`

    绘制圆角矩形。

    ```java
    canvas.drawRoundRect(left, top, right, bottom, radiusX, radiusY, paint);
    ```

- `drawCircle(float, float, float, Paint)`

    绘制圆。

    ```java
    canvas.drawCircle(circleX, circleY, radius, paint);
    ```

- `drawArc(float, float, float, float, float, float, boolean, Paint)`

    绘制弧形、扇形。绘制弧形与扇形的区分是第七个 `boolean` 参数。

    ```java
    // 绘制扇形
    canvas.drawArc(left, top, right, bottom, startAngle, sweepAngle, true, paint);
    // 绘制弧形
    canvas.drawArc(left, top, right, bottom, startAngle, sweepAngle, false, paint);
    ```

- `drawOval(float, float, float, float, Paint)`

    通过椭圆的外接矩形来绘制椭圆。

    ```java
    canvas.drawOval(left, top, right, bottom, paint);
    ```

- `drawText(String, float, float, Paint)`

    绘制文本。

    ```java
    canvas.drawText(text, startX, startY, paint);
    ```

- `drawPath(Path, Paint)`

    绘制路径。

    ```java
    Path path = new Path();
    path.moveTo(50, 50);
    path.lineTo(100, 100);
    path.lineTo(100, 300);
    path.lineTo(300, 50);
    canvas.drawPath(path, paint);
    ```

### XML 绘图

- `Bitmap`

    在 XML 中使用 `Bitmap` 代码如下所示。通过这样引用图片，可以将图片直接转成 `Bitmap` 在程序中使用。

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@mipmap/ic_launcher" />
    ```

- `Shape`

    通过 `Shape` 可以在 XML 中绘制各种形状。

    ```xml
    <!-- 默认为 rectangle -->
    <shape xmlns:android="http://schemas.android.com/apk/res/android"
        android:shape="[rectangle | oval | line | ring]" >

        <!-- 当 shape=“rectangle” 时使用 -->
            <!-- android:radius 半径，会被后面的单个半径属性覆盖。默认为 1dp -->
        <corners
            android:radius="integer"
            android:topLeftRadius="integer"
            android:topRightRadius="integer"
            android:bottomLeftRadius="integer"
            android:bottomRightRadius="integer" />

        <!-- 渐变 -->
        <gradient
            android:angle="integer"
            android:centerX="integer"
            android:centerY="integer"
            android:centerColor="integer"
            android:endColor="color"
            android:gradientRadius="integer"
            android:startColor="color"
            android:type="[linear | radial | sweep]"
            android:useLevel="[true | false]" />

        <padding
            android:left="integer"
            android:top="integer"
            android:right="integer"
            android:bottom="integer" />

        <!-- 指定大小。一般用在 imageview 配合 scaleType 属性使用 -->
        <size
            android:width="integer"
            android:height="integer" />

        <!-- 填充颜色 -->
        <solid
            android:color="color" />

        <!-- 指定边框 -->
            <!-- android:dashWidth 虚线宽度 -->
            <!-- android:dashGap 虚线间隔宽度 -->
        <stroke
            android:width="integer"
            android:color="color"
            android:dashWidth="integer"
            android:dashGap="integer" />
    </shape>
    ```

- `Layer`

    可以通过 `Layer` 实现图层的概念。

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <layer-list xmlns:android="http://schemas.android.com/apk/res/android">
        <!-- 图片1 -->
        <item android:drawable="@mipmap/ic_launcher" />

        <!-- 图片2 -->
        <item
            android:bottom="10dp"
            android:drawable="@mipmap/ic_launcher"
            android:left="10dp"
            android:right="10dp"
            android:top="10dp" />
    </layer-list>
    ```

- `Selector`

    `Selector` 的作用在于实现静态绘图中的事件反馈。通过给不同的事件设置不同的图像，从而在程序中根据用户输入，返回不同的效果。这一方法可以迅速制作 View 的触摸反馈。通过配置不同的触发事件， `Selector` 可以自动选择不同的图片。通常情况下，这些方法都可以共同作用。

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <selector xmlns:android="http://schemas.android.com/apk/res/android">
        <!-- 默认时的背景图片 -->
        <item android:drawable="@drawable/X1" />

        <!-- 没有焦点时的背景图片 -->
        <item android:drawable="@drawable/X2"
            android:state_window_focused="false" />

        <!-- 非触摸模式下获得焦点并单击时的背景图片 -->
        <item android:drawable="@drawable/X3"
            android:state_focused="true"
            android:state_pressed="true" />

        <!-- 触摸模式下单击时的背景图片 -->
        <item android:drawable="@drawable/X4"
            android:state_focused="false"
            android:state_pressed="true" />

        <!-- 选中时的图片背景 -->
        <item android:drawable="@drawable/X5"
            android:state_selected="true" />

        <!-- 获得焦点时的图片背景 -->
        <item android:drawable="@drawable/X6"
            android:state_focused="true" />
    </selector>
    ```

### 绘图技巧

#### `Canvas`

`Canvas` 作为绘制图形的直接对象，提供了以下几个方法。

- `Canvas#save()`

    将之前的所有已绘制图像保存，让后续的操作类似在一个新的图层上操作。

- `Canvas#restore()`

    将调用 `save()` 之后绘制的所有图像与调用 `save()` 之前的图像进行合并。

- `Canvas#translate(float, float)`

    将坐标系平移。调用 `translate(x, y)` 方法之后，将原点 `(0, 0)` 移动到 `(x, y)` 。之后的所有绘图操作都将以 `(x, y)` 为原点执行。

- `Canvas#rotate(float)`

    将坐标系翻转。将坐标系旋转一定的角度。

**例：** 创建一个仪表盘

```java
// 画外圆
Paint paintCircle = new Paint();
paintCircle.setStyle(Paint.Style.STROKE);
paintCircle.setAntiAlias(true);
paintCircle.setStrokeWidth(5);
canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth / 2, paintCircle);

// 画刻度
Paint painDegree = new Paint();
painDegree.setStrokeWidth(3);
for (int i = 0; i < 24; i++) {
    // 区分整点与非整点
    if (i == 0 || i == 6 || i == 12 || i == 18) {
        painDegree.setStrokeWidth(5);
        painDegree.setTextSize(30);
        canvas.drawLine(mWidth / 2, mHeight / 2 - mWidth / 2,
                mWidth / 2, mHeight / 2 - mWidth / 2 + 60, painDegree);
        String degree = String.valueOf(i);
        canvas.drawText(degree, mWidth / 2 - painDegree.measureText(degree) / 2,
                mHeight / 2 - mWidth / 2 + 90, painDegree);

    } else {
        painDegree.setStrokeWidth(3);
        painDegree.setTextSize(15);
        canvas.drawLine(mWidth / 2, mHeight / 2 - mWidth / 2,
                mWidth / 2, mHeight / 2 - mWidth / 2 + 30, painDegree);
        String degree = String.valueOf(i);
        canvas.drawText(degree, mWidth / 2 - painDegree.measureText(degree) / 2,
                mHeight / 2 - mWidth / 2 + 60, painDegree);
    }

    // 通过旋转画布简化坐标运算
    canvas.rotate(15, mWidth / 2, mHeight / 2);
}

// 画指针
Paint paintHour = new Paint();
paintHour.setStrokeWidth(20);
Paint paintMinute = new Paint();
paintMinute.setStrokeWidth(10);

canvas.save();
canvas.translate(mWidth / 2, mHeight / 2);
canvas.drawLine(0, 0, 100, 100, paintHour);
canvas.drawLine(0, 0, 100, 200, paintMinute);
canvas.restore();
```

#### `Layer`

一张复杂的图像可以由多个图层叠加形成一个图像。在 Android 中，使用 `saveLayer(float, float, float, float, Paint)` 方法创建一个图层，图层基于栈的结构进行管理。 Android 通过调用 `saveLayer(float, float, float, float, Paint)` 方法以及 `savaLayerAlpha(float, float, float, float, int)` 方法将一个图层入栈。使用 `restore()` 方法以及 `restoreToCount(int)` 方法将一个图层出栈。入栈时，后面所有的操作都发生在这个图层上。出栈时，会把图像绘制到上层 Canvas 上。

### 图像色彩特效处理

***TODO***

### 图形特效处理

***TODO***

### 画笔特效处理

***TODO***

### `SurfaceView`

`View` 可以满足大部分的绘图需求。 `View` 通过刷新来重绘视图。 Android 系统通过发出 `VSYNC` 信号进行屏幕的重绘，刷新的间隔时间为 16ms 。如果在 16ms 内 `View` 完成了所有操作，那么用户在视觉上，就不会产生卡顿的感觉。而如果执行的操作逻辑太多，特别是需要频繁刷新的界面上，就会不断阻塞主线程，从而导致画面卡顿。为了避免这一问题的产生， Android 系统提供了 `SurfaceView` 组件解决这个问题。 `SurfaceView` 与 `View` 的区别主要体现在以下几点：

- `View` 主要适用于主动更新的情况下，而 `SurfaceView` 主要适用于被动更新，例如频繁地刷新。
- `View` 在主线程中对画面进行刷新，而 `SurfaceView` 通常会通过一个子线程来进行页面的刷新。
- `View` 在绘图时没有使用双缓冲机制，而 `SurfaceView` 在底层实现机制中就已经实现了双缓冲机制。

如果自定义 View 需要频繁刷新，或者刷新时数据处理量比较大，可以考虑使用 `SurfaceView` 取代 `View` 。

通常情况下，使用以下步骤来创建 `SurfaceView` 模板。

- 创建 `SurfaceView`

    创建自定义 SurfaceView 继承自 `android.view.SurfaceView` ，重写构造方法。实现 `SurfaceHolder.Callback` 和 `Runnable` 两个接口并实现接口的方法。

- 初始化 `SurfaceView`

    在自定义的 SurfaceView 中，通常需要定义三个成员变量。然后需要初始化一个 `SurfaceHolder` 对象，并注册 `SurfaceHolder` 的回调方法。

- 使用 `SurfaceView`

    在 `surfaceCreated(SurfaceHolder)` 方法中开启子线程进行绘制。子线程使用 `while(mIsDrawing)` 循环不停地进行绘制。在绘制的具体逻辑中，通过 `SurfaceHolder` 对象的 `lockCanvas()` 方法，可以获得当前的 `Canvas` 绘图对象。在循环中会一直获取同一个 `Canvas` 对象。因此，之前的绘图操作都将被保留。如果需要擦除，可以在绘制前通过 `drawColor(int)` 方法进行清屏操作。绘制结束后通过 `mHolder.unlockCanvasAndPost(mCanvas)` 方法对画布内容进行提交。将语句放到 `finally` 代码块中，来保证每次都能将内容提交。

完整模版代码：

```java
public class SurfacePainter extends SurfaceView implements SurfaceHolder.Callback, Runnable {

    // SurfaceHolder
    private SurfaceHolder mHolder;
    // 用于绘图的 Canvas
    private Canvas mCanvas;
    // 子线程标志位
    private boolean mIsDrawing;

    public SurfacePainter(Context context) {
        this(context, null);
    }

    public SurfacePainter(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public SurfacePainter(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        mHolder = getHolder();
        mHolder.addCallback(this);

        // 其他的一些设置
        setFocusable(true);
        setFocusableInTouchMode(true);
        this.setKeepScreenOn(true);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }

    @Override
    public void run() {
        while (mIsDrawing) {
            drawContent();
        }
    }

    private void drawContent() {
        try {
            mCanvas = mHolder.lockCanvas();
            // 在 mCanvas 上进行绘图操作

        } catch (Exception e) {
            e.printStackTrace();

        } finally {
            if (mCanvas != null)
                mHolder.unlockCanvasAndPost(mCanvas);
        }
    }
```

**例：** 绘制一个正弦曲线。使用一个 `Path` 对象保存正弦函数上的坐标点，在子线程的 `while` 循环中，不断改变横纵坐标值。

```java
@Override
public void run() {
    while (mIsDrawing) {
        draw();
        x += 1;
        y = (int) (100 * Math.sin(x * 2 * Math.PI / 180) + 400);
        mPath.lineTo(x, y);
    }
}

private void draw() {
    try {
        mCanvas = mHolder.lockCanvas();
        // SurfaceView背景
        mCanvas.drawColor(Color.WHITE);
        mCanvas.drawPath(mPath, mPaint);

    } catch (Exception e) {
        e.printStackTrace();

    } finally {
        if (mCanvas != null){
            mHolder.unlockCanvasAndPost(mCanvas);
        }
    }
}
```

**例：** 实现一个简单的绘图板。在 `SurfaceView#onTouchEvent(MotionEvent)` 中记录 `Path` 路径，通过 `Path` 对象进行绘图。并在 `draw()` 方法中进行绘制。绘制不需要很频繁。因此我们可以在子线程中进行 `sleep` 操作，尽可能地节省系统资源。判断 `draw()` 方法所用逻辑时长确定 `sleep` 的时长， `sleep` 时间的取值一般在 50ms 到 100ms 左右。

> 此处书中示例代码有误。 `run()` 方法中代码应整体都在 `while` 循环内。

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    int x = (int) event.getX();
    int y = (int) event.getY();

    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            mPath.moveTo(x, y);
            break;

        case MotionEvent.ACTION_MOVE:
            mPath.lineTo(x, y);
            break;

        case MotionEvent.ACTION_UP:
            break;
    }

    return true;
}

@Override
public void run() {
    while (mIsDrawing) {
        long start = System.currentTimeMillis();
        draw();
        long end = System.currentTimeMillis();

        if (end - start < 100) {
            try {
                Thread.sleep(100 - (end - start));

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

private void draw() {
    try {
        mCanvas = mHolder.lockCanvas();
        mCanvas.drawColor(Color.WHITE);
        mCanvas.drawPath(mPath, mPaint);

    } catch (Exception e) {
        e.printStackTrace();

    } finally {
        if (mCanvas != null){
            mHolder.unlockCanvasAndPost(mCanvas);
        }
    }
}
```

[↑ 目录](#index)

------

<h2 id="chap7">Android 动画机制</h2>

### View 动画 `Animation`

`Animation` 框架定义了透明度、旋转、缩放和位移几种常见的动画，而且控制的是整个 `View` 。实现原理是每次绘制视图时 `View` 所在的 `ViewGroup` 中的 `drawChild(Canvas, View, long)` 函数获取该 `View` 的 `Transformation` 对象，然后调用 `canvas.concat(transformToApply.getMatrix())` 通过矩阵运算完成动画帧。如果动画没有完成，就继续调用 `invalidate()` 函数，启动下次绘制来驱动动画，从而完成整个动画的绘制。

视图动画提供了 `AlphaAnimation` ， `RotateAnimation` ， `TranslateAnimation` 和 `ScaleAnimation` 四种动画方式，并提供了 `AnimationSet` 动画集合混合使用多种动画。视图动画非常大的缺陷是不具备交互性。当某个元素发生视图动画后，其响应事件的位置还依然在动画开始前的地方，所以视图动画只能做普通的动画效果，避免交互的发生。但是它的优点也非常明显，即效率比较高且使用方便。

视图动画使用非常简单，不仅可以通过XML文件来描述一个动画过程，同样也可以使用代码来控制整个动画过程。

- `AlphaAnimation`

    为视图增加透明度的变换动画。

    ```java
    AlphaAnimation aa = new AlphaAnimation(fromAlpha, toAlpha);
    aa.setDuration(durationMillis);
    view.startAnimation(aa);
    ```

- `RotateAnimation`

    为视图增加旋转的变换动画。

    ```java
    RotateAnimation ra = new RotateAnimation(fromDegrees, toDegrees, pivotX, pivotY);
    ra.setDuration(durationMillis);
    view.startAnimation(ra);
    ```

    其参数分别为旋转的起始角度，结束角度和旋转中心点的 X Y 坐标。也可以通过设置参数来控制旋转动画的参考系。可取值为 `Animation.ABSOLUTE` ， `Animation.RELATIVE_TO_SELF` 或 `Animation.RELATIVE_TO_PARENT` 。

    ```java
    RotateAnimation ra = new RotateAnimation(fromDegrees, toDegrees, pivotXType, pivotXValue,
            pivotYType, pivotYValue);
    ra.setDuration(durationMillis);
    view.startAnimation(ra);
    ```

- `TranslateAnimation`

    为视图移动时增加位移动画。

    ```java
    TranslateAnimation ta = new TranslateAnimation(fromXDelta, toXDelta, fromYDelta, toYDelta);
    ta.setDuration(durationMillis);
    view.startAnimation(ta);
    ```

- `ScaleAnimation`

    为视图的缩放增加动画效果。

    ```java
    ScaleAnimation sa = new ScaleAnimation(fromX, toX, fromY, toY);
    sa.setDuration(durationMillis);
    view.startAnimation(sa);
    ```

    与旋转动画一样，缩放动画也可以设置缩放的中心点。

    ```java
    ScaleAnimation sa = new ScaleAnimation(fromX, toX, fromY, toY,
            pivotXType, pivotXValue, pivotYType, pivotYValue);
    sa.setDuration(durationMillis);
    view.startAnimation(sa);
    ```

- `AnimationSet`

    通过 `AnimationSet` ，可以将动画以组合的形式展现出来。

    ```java
    AnimationSet as = new AnimationSet(shareInterpolator);
    as.setDuration(durationMillis);

    AlphaAnimation aa = new AlphaAnimation(fromAlpha, toAlpha);
    aa.setDuration(durationMillis);
    as.addAnimation(aa);

    TranslateAnimation ta = new TranslateAnimation(fromXDelta, toXDelta, fromYDelta, toYDelta);
    ta.setDuration(durationMillis);
    as.addAnimation(ta);

    view.startAnimation(as);
    ```

对于动画事件，Android 也提供对应的监听回调，通过监听回调，可以获取到动画的开始、结束和重复事件，并针对相应的事件做出不同的处理。要添加相应的监听方法，代码如下所示。

```java
animation.setAnimationListener(new Animation.AnimationListener() {
    @Override
    public void onAnimationStart(Animation animation) {
    }

    @Override
    public void onAnimationEnd(Animation animation) {
    }

    @Override
    public void onAnimationRepeat(Animation animation) {
    }
});

```

### 属性动画 `Animator`

动画框架 `Animation` 存在一些局限性，动画改变的只是显示，并不能响应事件。因此，在 Android 3.0 之后， Google 提出了属性动画帮助开发者实现更加丰富的动画效果。

在 `Animator` 框架中使用最多的是 `AnimatorSet` 和 `ObjectAnimator` 配合。使用 `ObjectAnimator` 进行更精细化控制，只控制一个对象的一个属性值。使用多个 `ObjectAnimator` 组合到 `AnimatorSet` 形成一个动画。而且 `ObjectAnimator` 能够自动驱动，可以调用 `setFrameDelay(longframeDelay)` 设置动画帧之间的间隙时间，调整帧率，减少动画过程中频繁绘制界面，而在不影响动画效果的前提下减少 CPU 资源消耗。最重要的是，属性动画通过调用属性的 get ， set 方法真实地控制了一个 View 的属性值，因此强大的属性动画框架，基本可以实现所有的动画效果。

- `ObjectAnimator`

    创建 `ObjectAnimator` 需通过他的静态工厂类直接返回 `ObjectAnimator` 对象。参数包括一个对象和对象的属性名字。这个属性必须有 `get` 和 `set` 函数，内部会通过 Java 反射机制来调用 `set` 函数修改对象属性值。也可以调用 `setInterpolator(TimeInterpolator)` 设置相应的差值器。

    ```java
    ObjectAnimator animator = ObjectAnimator.ofFloat(target, propertyName, values);
    animator.setDuration(durationMillis);
    animator.start();
    ```

    通过 `ObjectAnimator` 的静态工厂方法，创建 `ObjectAnimator` 对象。第一个参数是需要操纵的 View 。第二个参数是要操纵的属性。最后一个参数是一个可变数组参数，需要传入该属性变化的取值过程。也可以给属性动画设置显示时长、差值器等属性，这些参数与在视图动画中的设置方法类似。

    在使用 `ObjectAnimator` 的时候，有一点非常重要。要操纵的属性必须具有 `get` 和 `set` 方法，不然 `ObjectAnimator` 就无法起效。下面就是一些常用的可以直接使用属性动画的属性值。

  - `translationX` 和 `translationY`

    这两个属性作为一种增量控制 `View` 对象从它布局容器的左上角坐标开始的位置。

  - `rotation` `rotationX` 和 `rotationY`

    这三个属性控制 `View` 对象围绕支点进行 2D 和 3D 旋转。

  - `scaleX` 和 `scaleY`

    这两个属性控制 `View` 对象围绕它的支点进行 2D 缩放。

  - `pivotX` 和 `pivotY`

    这两个属性控制 `View` 对象的支点位置，围绕这个支点进行旋转和缩放变换处理。默认情况下，该支点的位置就是 `View` 对象的中心点。

  - `x` 和 `y`

    这是两个简单实用的属性。它描述了 `View` 对象在它的容器中的最终位置，它是最初的左上角坐标和 `translationX` `translationY` 值的累计和。

  - `alpha`

    它表示 `View` 对象的 alpha 透明度。默认值是 1 不透明， 0 代表完全透明。

  如果一个属性没有 `get` `set` 方法，有两种方案来解决这个问题。一个是通过自定义一个属性类或者包装类，来间接地给这个属性增加 `get` `set` 方法。或者通过 `ValueAnimator` 来实现。这里看看如何使用包装类的方法给一个属性增加 `get` `set` 方法，代码如下所示。

    ```java
    private static class WrapperView {
        private View mTarget;
        public WrapperView(View target) {
            mTarget = target;
        }

        public int getWidth() {
            return mTarget.getLayoutParams().width;
        }

        public void setWidth(int width) {
            mTarget.getLayoutParams().width = width;
            mTarget.requestLayout();
        }
    }
    ```

    使用时只需要操纵包装类就可以间接调用到 `get` `set` 方法了，代码如下所示。

    ```java
    ViewWrapper wrapper = new ViewWrapper(targetView);
    ObjectAnimator
            .ofInt(wrapper, "width", values)
            .setDuration(durationMillis)
            .start();
    ```

- `ValueAnimator`

    `ValueAnimator` 是属性动画的核心， `ObjectAnimator` 也是继承自 `ValueAnimator` 。`ValueAnimator` 本身不提供任何动画效果。它像一个数值发生器，用来产生具有一定规律的数字，从而让调用者来控制动画的实现过程。 `ValueAnimator` 的一般使用方法为在 `ValueAnimator` 的 `AnimatorUpdateListener` 中监听数值的变换，从而完成动画的变换。

    ```java
    ValueAnimator animator = ValueAnimator.ofFloat(values);
    animator.setTarget(view);
    animator.setDuration(durationMillis);
    animator.start();
    animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
        @Override
        public void onAnimationUpdate(ValueAnimator animation) {
            Float value = (Float) animation.getAnimatedValue();
            // use the value
        }
    });
    ```

- `PropertyValuesHolder`

    在属性动画中，如果针对同一个对象的多个属性同时作用多种动画，可以使用 `PropertyValuesHolder` 来实现。比如平移动画，如果需要在平移的过程中，同时改变 X Y 轴的缩放。分别使用 `PropertyValuesHolder` 对象来控制 `translationX` `scaleX` `scaleY` 三个属性，最后调用 `ObjectAnimator.ofPropertyValuesHolder(Object, PropertyValuesHolder...)` 方法实现多属性动画的共同作用。代码如下所示。

    ```java
    PropertyValuesHolder pvh1 = PropertyValuesHolder.ofFloat("translationX", values1);
    PropertyValuesHolder pvh2 = PropertyValuesHolder.ofFloat("scaleX", values2);
    PropertyValuesHolder pvh3 = PropertyValuesHolder.ofFloat("scaleY", values3);
    ObjectAnimator
            .ofPropertyValuesHolder(view, pvh1, pvh2, pvh3)
            .setDuration(durationMillis)
            .start();
    ```

- `AnimatorSet`

    对于一个属性同时作用多个属性动画效果，可以使用 `PropertyValuesHolder` 实现。但 `AnimatorSet` 不仅能实现这样的效果，同时也能实现更为精确的顺序控制。在属性动画中， `AnimatorSet` 通过 `playTogether(Animator...)` ， `playSequentially(Animator...)` ， `play(Animator).with(Animator)` ， `play(Animator).before(Animator)` ， `play(Animator).after(Animator)` 这些方法控制多个动画的协同工作，从而做到对动画播放顺序的精确控制。如果使用 `AnimatorSet` 实现上面的动画效果，代码如下所示。

    ```java
    ObjectAnimator animator1 = ObjectAnimator.ofFloat(view, "translationX", values1);
    ObjectAnimator animator2 = ObjectAnimator.ofFloat(view, "scaleX", values2);
    ObjectAnimator animator3 = ObjectAnimator.ofFloat(view, "scaleY", values3);
    AnimatorSet set = new AnimatorSet();
    set.setDuration(durationMillis);
    set.playTogether(animator1, animator2, animator3);
    set.start();
    ```

- `AnimatorListener`

    一个完整的动画具有 `Start` `Repeat` `End` `Cancel` 四个过程，通过 Android 提供的接口，可以很方便地监听到这四个事件，代码如下所示。

    ```java
    ObjectAnimator anim = ObjectAnimator.ofFloat(view, "alpha", values);

    anim.addListener(new AnimatorListener() {
        @Override
        public void onAnimationStart(Animator animation) {
        }

        @Override
        public void onAnimationRepeat(Animator animation) {
        }

        @Override
        public void onAnimationEnd(Animator animation) {
        }

        @Override
        public void onAnimationCancel(Animator animation) {
        }
    });

    anim.start();
    ```

    大部分的时候，我们只关心 `onAnimationEnd` 事件，所以 Android 也提供了一个 `AnimatorListenerAdapter` 让我们选择必要的事件进行监听，代码如下所示。

    ```java
    anim.addListener(new AnimatorListenerAdapter() {
        @Override
        public void onAnimationEnd(Animator animation) {
        }
    });
    ```

- `XML` 定义

    属性动画同视图动画一样，也可以直接写在 XML 文件中，写法很相似。代码如下所示。

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
        android:duration="1000"
        android:propertyName="scaleX"
        android:valueFrom="1.0"
        android:valueTo="2.0"
        android:valueType="floatType" />
    ```

    在程序中使用XML定义的属性动画也非常简单，代码如下所示。

    ```java
    Animator anim = AnimatorInflater.loadAnimator(context, R.animator.scalex);
    anim.setTarget(view);
    anim.start();
    ```

- `View#animate()`

    `View#animate()` 方法可以直接驱动属性动画，代码如下所示。

    ```java
    view.animate()
            .alpha(valueAlpha)
            .y(valueY)
            .setDuration(durationMillis)
            .withStartAction(new Runnable() {
                @Override
                public void run() {
                }
            })
            .withEndAction(new Runnable() {
                @Override
                public void run() {
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                        }
                    });
                }
            }).start();
    ```

### 布局动画

布局动画指作用在 `ViewGroup` 上，给 `ViewGroup` 增加 `View` 时添加一个动画过渡效果。最简单的布局动画是在 `ViewGroup` 的 XML 中，使用以下代码来打开布局动画。当 `ViewGroup` 添加 `View` 时，子 View 会呈现逐渐显示的过渡效果。不过这个效果是 Android 默认的显示的过渡效果，且无法使用自定义的动画来替换这个效果。

```xml
android:animateLayoutChanges="true"
```

另外，还可以使用 `LayoutAnimationController` 类自定义一个子 View 的过渡效果，代码如下所示。给 `LinearLayout` 增加了一个视图动画，让子 View 在出现的时候，有一个缩放动画效果。

```java
LinearLayout ll = (LinearLayout) findViewById(R.id.ll);
// 设置过渡动画
ScaleAnimation sa = new ScaleAnimation(fromX, toX, fromY, toY);
sa.setDuration(durationMillis);
// 设置布局动画的显示属性
LayoutAnimationController lac = new LayoutAnimationController(sa, delay);
lac.setOrder(LayoutAnimationController.ORDER_NORMAL);
// 为 ViewGroup 设置布局动画
ll.setLayoutAnimation(lac);
```

`LayoutAnimationController` 的第一个参数是需要作用的动画。第二个参数是每个子 View 显示的延迟时间。当延迟时间不为 0 时，可以设置子 View 显示的顺序，如下所示。

- `LayoutAnimationController.ORDER_NORMAL`  —— 顺序
- `LayoutAnimationController.ORDER_RANDOM`  —— 随机
- `LayoutAnimationController.ORDER_REVERSE` —— 反序

### 插值器 `Interpolators`

插值器是动画中一个非常重要的概念。通过插值器 `Interpolators` ，可以定义动画变换速率。这一点非常类似物理中的加速度，其作用主要是控制目标变量的变化值进行对应的变化。同样的一个动画变换起始值，在不同的插值器作用下，每个单位时间内所达到的变化值也是不一样的。例如一个位移动画，如果使用线性插值器，那么在持续时间内，单位时间所移动的距离都是一样的；如果使用加速度插值器，那么单位时间内所移动的距离将越来越快。

### 自定义动画

创建自定义动画只需要实现它的 `applyTransformation(float, Transformation)` 方法。该方法第一个 `float` 参数是插值器的时间因子，这个因子是由动画当前完成的百分比和当前时间所对应的插值计算所得，取值范围为 0.0 到 1.0 。第二个 `Transformation` 参数是矩阵的封装类，一般使用这个类来获得当前的矩阵对象。通过改变获得的 `Matrix` 对象，可以实现动画效果。对于 `Matrix` 的变换操作，基本可以实现任何效果的动画。

```java
@Override
protected void applyTransformation(float interpolatedTime, Transformation t) {
    final Matrix matrix = t.getMatrix();
    // 通过 matrix 的各种操作来实现动画
}
```

通常情况下，还需要覆盖父类的 `initialize(int, int, int, int)` 方法实现一些初始化工作。

**例：** 模拟电视机关闭的效果。

要实现电视机关闭的效果，让一个图片纵向比例不断缩小即可。其中 `mCenterWidth` ， `mCenterHeight` 即为缩放的中心点，设置为图片中心即可。

```java
@Override
protected void applyTransformation(float interpolatedTime, Transformation t) {
    final Matrix matrix = t.getMatrix();
    matrix.preScale(1, 1 - interpolatedTime, mCenterWidth, mCenterHeight);
}
```

还可以设置更精确的插值器，并将 0.0 到 1.0 的时间因子拆分成不同的过程，从而对不同的过程采用不同的动画效果，模拟更加真实的特效。

**例：** 使用 `android.graphics.Camera` 类实现自定义 3D 动画效果。

`Camera` 类封装了 openGL 的 3D 动画，从而可以非常方便地创建 3D 动画效果。把 `Camera` 想象成一个真实的摄像机，当物体固定在某处时，只要移动摄像机就能拍摄到具有立体感的图像，因此通过它可以实现各种 3D 效果。

首先，在初始化方法中对 `Camera` 和其他参数进行初始化，代码如下。

```java
@Override
public void initialize(int width, int height, int parentWidth, int parentHeight) {
    super.initialize(width, height, parentWidth, parentHeight);
    // 设置默认时长
    setDuration(durationMillis);
    // 动画结束后保留状态
    setFillAfter(true);
    // 设置默认插值器
    setInterpolator(new BounceInterpolator());
    mCenterWidth = width / 2;
    mCenterHeight = height / 2;
}
```

接下来，定义动画的进行过程。

```java
@Override
protected void applyTransformation(float interpolatedTime, Transformation t) {
    final Matrix matrix = t.getMatrix();
    mCamera.save();
    // 使用 Camera 设置旋转的角度
    mCamera.rotateY(mRotateY * interpolatedTime);
    // 将旋转变换作用到 matrix 上
    mCamera.getMatrix(matrix);
    mCamera.restore();
    // 通过 pre 方法设置矩阵作用前的偏移量来改变旋转中心
    matrix.preTranslate(mCenterWidth, mCenterHeight);
    matrix.postTranslate(-mCenterWidth, -mCenterHeight);
}
```

### SVG 矢量动画

可伸缩矢量图形 (Scalable Vector Graphics) 是用于网络的基于矢量的图形。其使用 XML 格式定义图形，图像在放大或改变尺寸的情况下其图形质量不会有所损失。它是万维网联盟的标准，与诸如 `DOM` 和 `XSL` 之类的 W3C 标准是一个整体。 Android 在 5.0 之后添加了对 `SVG` 的 `<path>` 标签的支持。

#### SVG 标签与指令

使用 `<path>` 标签创建 SVG ，就像用指令的方式来控制一只画笔。 `<path>` 标签所支持的指令有以下几种。

- `M X Y` = moveTo(x ,y)

    将画笔移动到指定的坐标位置，但不发生绘制。

- `L X Y` = lineTo(x, y)

    绘制直线的指令，画直线到指定的坐标位置。参数是一个点坐标，如 `L 200 400` 绘制直线。

- `H X` = horizontalLineTo(x)

    画水平线到指定的 X 坐标位置。

- `V Y` = verticalLineTo(y)

    画垂直线到指定的 Y 坐标位置。

- `C X1 Y1 X2 Y2 ENDX ENDY` = curveTo(x1, y1, x2, y2, endX, endY)

    三次贝赛曲线。

- `S X2 Y2 ENDX ENDY` = smoothCurveTo(x2, y2, endX, endY)

    三次贝赛曲线。

- `Q X Y ENDX ENDY` = quadraticBelzierCurve(x, y, endX, endY)

    二次贝赛曲线。

- `T ENDX ENDY` = smoothQuadraticBelzierCurveto(endX, endY)

    映射前面路径后的终点。

- `A RX RY XROTATION FLAG1 FLAG2 X Y` = ellipticalArc(rX, rY, xRotation, flag1, flag2, x, y)

    绘制一段弧线，且允许弧线不闭合。可以把 `A` 命令绘制的弧线想象成是椭圆的某一段。 `A` 指令以下有七个参数。

  - `RX` 与 `RY`

    指所在椭圆的半轴大小。

  - `XROTATION`

    指椭圆的 X 轴与水平方向顺时针方向夹角，可以想象成一个水平的椭圆绕中心点顺时针旋转 `XROTATION` 的角度。

  - `FLAG1`

    只有两个值。 1 表示大角度弧线， 0 为小角度弧线。

  - `FLAG2`

    只有两个值，确定从起点至终点的方向。 1 为顺时针， 0 为逆时针。

  - `X`，`Y`

    终点坐标。

- `Z` = closePath()

    关闭路径。

在使用上面的指令时，需要注意以下几点。

- 坐标轴以 `(0,0)` 为中心。 X 轴水平向右， Y 轴水平向下。
- 所有指令大小写均可。大写绝对定位，参照全局坐标系。小写相对定位，参照父容器坐标系。
- 指令和数据间的空格可以省略。
- 同一指令出现多次可以只用一个。

#### SVG 编辑器

[SVG 在线编辑器](http://editor.method.ac/ "SVG Editor")

通过可视化编辑好图形后，点击 `View` → `Source` 可以转换为 SVG 代码。

#### 在 Android 中使用

Android 中提供了两个 API 帮助支持 SVG 。其中 `VectorDrawable` 可以创建基于 XML 的 SVG 图形。 `AnimatedVectorDrawable` 可以实现动画效果。

- `VectorDrawable`

    在 XML 中创建静态的 SVG 图形，通常会形成树形结构。 `path` 是 SVG 树形结构中的最小单位，而通过 `group` 可以将不同的 `path` 进行组合。

    在 XML 中创建 SVG 图形，首先需要在 XML 中通过 `<vector>` 标签来声明对 SVG 的使用。其中包含两组宽高属性 `height` 与 `width` 和 `viewportHeight` 与 `viewportWidth` 。这两组属性分别具有不同的含义。 `height` 与 `width` 表示该 SVG 图形的具体大小，而 `viewportHeight` 与 `viewportWidth` 表示 SVG 图形划分的比例。后面在绘制 path 时所使用的参数，就是根据这两个值来进行转换的。例如，将 200dp 划分为 100 份，在绘制图形时使用坐标 `(50,50)` ，则意味着该坐标位于该 SVG 图形正中间。因此，`height` 和 `width` 的比例与 `viewportHeight` 和 `viewportWidth` 的比例，必须保持一致，不然图形就会发生压缩、形变。接下来通过添加 `<group>` 标签和 `<path>` 标签绘制一个 SVG 图形，其中 `pathData` 属性是绘制 SVG 图形所用到的指令。

    ```xml
    <vector xmlns:android="http://schemas.android.com/apk/res/android"
        android:height="200dp"
        android:width="200dp"
        android:viewportHeight="100"
        android:viewportWidth="100">
            <group
                android:name="test"
                android:rotation="0">

            <path
                android:fillColor="@android:color/holo_blue_light"
                android:pathData="
                M 25 50
                a 25,25 0 1,0 50,0" />
            </group>
    </vector>
    ```

    这里使用了 `android:fillColor` 属性绘制图形，因此绘制出来的是一个填充的图形。如果要绘制一个非填充的图形，可以使用以下属性。

    ```xml
    android:strokeColor="@android:color/holo_blue_light"
    android:strokeWidth="2"
    ```

- `AnimatedVectorDrawable`

    `AnimatedVectorDrawable` 的作用是给 `VectorDrawable` 提供动画效果。 Google 的工程师将 `AnimatedVectorDrawable` 比喻为一个胶水，通过 `AnimatedVectorDrawable` 来连接静态的 `VectorDrawable` 和动态的 `objectAnimator` 。

    使用 `AnimatedVectorDrawable` 实现 SVG 图形的动画效果，首先在 XML 文件中通过 `<animated-vector>` 标签来声明对 `AnimatedVectorDrawable` 的使用，并指定其作用的 `path` 或 `group` 。 `target` 中指定的的 `name` 属性，必须与 `VectorDrawable` 中需要作用的 `name` 属性保持一致，这样系统才能找到要实现动画的元素。最后，通过 `target` 的 `animation` 属性，将一个动画作用到对应 `name` 的元素上。

    动画效果的实现，是通过属性动画来实现的，只是属性稍有不同。在 `<group>` 标签和 `<path>` 标签中添加 `rotation` ， `fillColor` 或 `pathData` 等属性，在 `objectAnimator` 中就可以通过指定 `android:propertyName` 属性来选择控制哪一种属性。通过 `android:valueFrom` 和 `android:valueTo` 属性，可以控制动画的关键值。需要注意的是，如果控制属性 `pathData` ，需要添加一个属性 `android:valueType="pathType"` 告诉系统进行 `pathData` 变换。

    ```xml
    <animated-vector
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:drawable="@drawable/vector" >

            <target
            android:name="test"
            android:animation="@anim/anim_path1" />
    </animated-vector>
    ```

    当 XML 文件准备好，就可以在代码中控制 SVG 动画。可以将 `AnimatedVectorDrawable` XML 文件设置给一个 `ImageView` 显示。然后在程序中使用以下代码开始 SVG 动画。

    ```java
    ((Animatable) imageView.getDrawable()).start();
    ```

#### SVG 动画示例

**例：** 线图动画。图像初始状态为两条平行的水平线条。开始 SVG 动画后，上下两根线条会从中间折断并向中间折起，最终形成一个 X 。

首先创建静态 SVG 图形，即静态的 `VectorDrawable` 。 `path1` 和 `path2` 分别绘制了两条直线的初始状态。每条直线由三个点控制，即起始点和中点，形成初始状态。

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="200dp"
    android:height="200dp"
    android:viewportHeight="100"
    android:viewportWidth="100">

    <group>
        <path
            android:name="path1"
            android:pathData="
                M 20,80
                L 50,80 80,80"
            android:strokeColor="@android:color/holo_green_dark"
            android:strokeLineCap="round"
            android:strokeWidth="5" />

        <path
            android:name="path2"
            android:pathData="
                M 20,20
                L 50,20 80,20"
            android:strokeColor="@android:color/holo_green_dark"
            android:strokeLineCap="round"
            android:strokeWidth="5" />
    </group>
</vector>
```

接下来实现变换的 `objectAnimator` 动画。定义 `pathType` 的属性动画，并指定变换的关键值。需要注意，在 SVG 的路径变换属性动画中，变换前后的节点数必须相同。这就是前面需要使用三个点来绘制一条直线的原因，因为后面需要中点进行动画变换。

```xml
<!-- anim_path1 -->
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="500"
    android:interpolator="@android:anim/bounce_interpolator"
    android:propertyName="pathData"
    android:valueFrom="
        M 20,80
        L 50,80 80,80"
    android:valueTo="
        M 20,80
        L 50,50 80,80"
    android:valueType="pathType" />
```

```xml
<!-- anim_path2 -->
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="500"
    android:interpolator="@android:anim/bounce_interpolator"
    android:propertyName="pathData"
    android:valueFrom="
        M 20,20
        L 50,20 80,20"
    android:valueTo="
        M 20,20
        L 50,50 80,20"
    android:valueType="pathType" />
```

有了 `VectorDrawable` 和 `objectAnimator` ，剩下的只需要使用 `AnimatedVectorDrawable` 来将它们黏合在一起。

```xml
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/trick">

    <target
        android:name="path1"
        android:animation="@animator/anim_path1" />

    <target
        android:name="path2"
        android:animation="@animator/anim_path2" />
</animated-vector>
```

最后只要在代码中启动动画即可。

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    imageView = (ImageView) findViewById(R.id.image);
    imageView.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            animate();
        }
    });
}

private void animate() {
    Drawable drawable = imageView.getDrawable();
    if (drawable instanceof Animatable) {
        ((Animatable) drawable).start();
    }
}
```

**例：** 模拟三球仪。三球仪是天文学中一个星象仪器，用来模拟地、月、日三个星体的绕行轨迹。即地球绕太阳旋转，月球绕地球旋转的同时，绕太阳旋转。

首先绘制一个静态的地、月、日三星系统。在 `sun` 这个 `group` 中，有一个 `earth` 的 `group` 。同时使用 `android:pivotX` 和 `android:pivotY` 属性设置其旋转中心。这两个 `group` 分别代表了太阳系系统和地月系系统。

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="200dp"
    android:height="200dp"
    android:viewportHeight="100"
    android:viewportWidth="100">

    <group
        android:name="sun"
        android:pivotX="60"
        android:pivotY="50"
        android:rotation="0">

        <path
            android:name="path_sun"
            android:fillColor="@android:color/holo_red_light"
            android:pathData="
                M 50,50
                a 10,10 0 1,0 20,0
                a 10,10 0 1,0 -20,0" />

        <group
            android:name="earth"
            android:pivotX="75"
            android:pivotY="50"
            android:rotation="0">

            <path
                android:name="path_earth"
                android:fillColor="@android:color/holo_blue_light"
                android:pathData="
                    M 70,50
                    a 5,5 0 1,0 10,0
                    a 5,5 0 1,0 -10,0" />

            <group>
                <path
                    android:fillColor="@android:color/darker_gray"
                    android:pathData="
                        M 90,50
                        m -5 0
                        a 4,4 0 1,0 8 0
                        a 4,4 0 1,0 -8,0" />
            </group>
        </group>
    </group>
</vector>
```

下面对这两个 `group` 分别进行 SVG 动画。为了简化示例，我们让两个动画效果的实现完全相同。

```xml
<!-- anim_sun -->
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="4000"
    android:propertyName="rotation"
    android:valueFrom="0"
    android:valueTo="360" />
```

```xml
<!-- anim_earth -->
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="4000"
    android:propertyName="rotation"
    android:valueFrom="0"
    android:valueTo="360" />
```

最后，使用 `AnimatedVectorDrawable` 黏合 SVG 静态图形和动画。

```xml
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/solar">

    <target
        android:name="sun"
        android:animation="@animator/anim_sun" />

    <target
        android:name="earth"
        android:animation="@animator/anim_earth" />
</animated-vector>
```

启动动画的方法与之前的实例相同，这里不再赘述。

**例：** 轨迹动画。使用 `trimPathStart` 动画，可以像画出一个图形一样来进行绘制，从而形成一个轨迹动画效果。

首先需要用 SVG 绘制一个搜索栏，代码如下所示。

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="160dp"
    android:height="30dp"
    android:viewportHeight="30"
    android:viewportWidth="160">

    <path
        android:name="search"
        android:pathData="M 141,17
            A 9,9 0 1,1 142,16
            L 149,23"
        android:strokeAlpha="0.8"
        android:strokeColor="#ff3570be"
        android:strokeLineCap="round"
        android:strokeWidth="2" />

    <path
        android:name="bar"
        android:pathData="M 0,23
            L 149,23"
        android:strokeAlpha="0.8"
        android:strokeColor="#ff3570be"
        android:strokeLineCap="square"
        android:strokeWidth="2" />
</vector>
```

下面利用属性动画实现动画效果。将 `propertyName` 设置为 `trimPathStart` 。这个属性就是利用 0 到 1 的百分比来按照轨迹绘制 SVG 图像。类似的，还有 `trimPathEnd` 这个属性，实现代码如下所示。

```xml
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:interpolator="@android:interpolator/accelerate_decelerate"
    android:propertyName="trimPathStart"
    android:valueFrom="0"
    android:valueTo="1"
    android:valueType="floatType" />
```

最后，使用 `AnimatedVectorDrawable` 黏合 SVG 静态图形和动画。

```xml
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/search_bar">

    <target
        android:name="search"
        android:animation="@animator/anim_search" />
</animated-vector>
```

启动动画的方法与之前的实例相同，这里不再赘述。

### 动画示例

**例：** 灵动菜单。当用户点击小红点后，弹出菜单，并带有一个缓冲的过渡动画。它具有用户交互性，所以必须使用属性动画。只需要针对每个不同的按钮设置不同的动画，并设置相应的差值器就可以实现展开、合拢效果了。

```java
private void startAnim() {
    ObjectAnimator anim0 = ObjectAnimator.ofFloat(mImageViews.get(0), "alpha", 1f, 0.5f);
    ObjectAnimator anim1 = ObjectAnimator.ofFloat(mImageViews.get(1), "translationY", 200f);
    ObjectAnimator anim2 = ObjectAnimator.ofFloat(mImageViews.get(2), "translationX", 200f);
    ObjectAnimator anim3 = ObjectAnimator.ofFloat(mImageViews.get(3), "translationY", -200f);
    ObjectAnimator anim4 = ObjectAnimator.ofFloat(mImageViews.get(4), "translationX", -200f);
    AnimatorSet set = new AnimatorSet();
    set.setDuration(500);
    set.setInterpolator(new BounceInterpolator());
    set.playTogether(anim0, anim1, anim2, anim3, anim4);
    set.start();
}
```

为按钮设置点击事件即可完成整个功能。

```java
@Override
public void onClick(View v) {
    switch (v.getId()) {
        case R.id.imageView0:
            startAnim();
            break;
    }
}
```

**例：** 计时器动画。当用户点击后，数字会不断增加。实现计时器动画效果的方法有很多，这里为了演示 `ValueAnimator` 的效果，因而使用 `ValueAnimator` 实现。

完成效果只需要借助 `ValueAnimator` 实现数字的不断增加，并将值设置给 `TextView` 即可。

```java
ValueAnimator valueAnimator = ValueAnimator.ofInt(0, 100);
valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator animation) {
        tv.setText(animation.getAnimatedValue());
    }
});
valueAnimator.setDuration(3000);
valueAnimator.start();
```

**例：** 下拉展开动画。当点击一个 View 的时候，显示下面隐藏的一个 View 。要实现这个功能，需要将 View 的 `visibility` 属性由 `gone` 设置为 `visible` 即可，但是这个过程是瞬间完成的，还需要在 View 显示时增加一个动画效果。使用 `ValueAnimator` 模拟这样的效果，让隐藏的 View 的高度不断发生变化而不是一下增大到目标值。

首先，写一个简单的布局。两个 `LinearLayout` 一个显示，一个隐藏。

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:background="@android:color/holo_blue_bright"
        android:gravity="center_vertical"
        android:onClick="llClick"
        android:orientation="horizontal">

        <ImageView
            android:id="@+id/app_icon"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:src="@mipmap/ic_launcher" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="5dp"
            android:gravity="left"
            android:text="Click Me"
            android:textSize="30sp" />
    </LinearLayout>

    <LinearLayout
        android:id="@+id/hidden_view"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:background="@android:color/holo_orange_light"
        android:gravity="center_vertical"
        android:orientation="horizontal"
        android:visibility="gone">

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:src="@mipmap/ic_launcher" />

        <TextView
            android:id="@+id/tv_hidden"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:gravity="center"
            android:text="I am hidden"
            android:textSize="20sp" />
    </LinearLayout>
</LinearLayout>
```

当点击上面的 `LinearLayout` 时，需要获取到隐藏的 `LinearLayout` 最终需要到达的高度，即我们的目标值。通过将布局文件中的 dp 值转化为像素值即可。然后给这个过程增加动画效果。需要使用 `ValueAnimator` 来创建一个从 0 到目标值的数值发生器，并由此来改变 View 的布局属性。通过这样一个 `ValueAnimator` 就可以实现显示、隐藏的动画效果了。完整代码如下所示。

```java
public class AnimActivity extends AppCompatActivity {

    private LinearLayout mHiddenView;
    private float mDensity;
    private int mHiddenViewMeasuredHeight;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_anim);
        mHiddenView = (LinearLayout) findViewById(R.id.hidden_view);
        // 获取像素密度
        mDensity = getResources().getDisplayMetrics().density;
        // 获取布局的高度
        mHiddenViewMeasuredHeight = (int) (mDensity * 40 + 0.5);
    }

    public void llClick(View view) {
        if (mHiddenView.getVisibility() == View.GONE) {
            // 打开动画
            animateOpen(mHiddenView);
        } else {
            // 关闭动画
            animateClose(mHiddenView);
        }
    }

    private void animateOpen(final View view) {
        view.setVisibility(View.VISIBLE);
        ValueAnimator animator = createDropAnimator(view, 0, mHiddenViewMeasuredHeight);
        animator.start();
    }

    private void animateClose(final View view) {
        int origHeight = view.getHeight();
        ValueAnimator animator = createDropAnimator(view, origHeight, 0);
        animator.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                view.setVisibility(View.GONE);
            }
        });
        animator.start();
    }

    private ValueAnimator createDropAnimator(final View view, int start, int end) {
        ValueAnimator animator = ValueAnimator.ofInt(start, end);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                int value = (Integer) valueAnimator.getAnimatedValue();
                ViewGroup.LayoutParams layoutParams = view.getLayoutParams();
                layoutParams.height = value;
                view.setLayoutParams(layoutParams);
            }
        });
        return animator;
    }
}
```

[↑ 目录](#index)

------

<h2 id="chap8">Activity 与 Activity 调用栈</h2>

### Activity

`Activity` 作为四大组件中出现频率最高的组件，我们在 Android 的各个地方都能看见它的影子。`Activity` 是与用户交互的第一接口，它提供了一个用户完成指令的窗口。当开发者创建 `Activity` 之后，通过调用 `setContentView(View)` 方法给该 `Activity` 指定一个界面布局，并以此为基础提供给用户交互的接口。系统采用 `Activity 栈` 的方式来管理 `Activity` 。

`Activity` 一个特点就是拥有多种形态。它可以在多种形态间进行切换，以此来控制自己的生命周期。

- `Active/Running`

    此时 `Activity` 处于 `Activity 栈` 的最顶层。可见，并与用户进行交互。

- `Paused`

    当 `Activity` 失去焦点，被一个在栈顶的非全屏的 `Activity` 或一个透明的 `Activity` 覆盖时， `Activity` 就转化为 `Paused` 形态。但它只是失去了与用户交互的能力，所有状态信息、成员变量都还保持。只有在系统内存极低的情况下，才会被系统回收掉。

- `Stopped`

    如果一个 `Activity` 被另一个 `Activity` 完全覆盖，那么 `Activity` 就会进入 `Stopped` 形态。此时它不再可见，但却依然保持了所有状态信息和成员变量。

- `Killed`

    当 `Activity` 被系统回收掉或者 `Activity` 从来没有创建过， `Activity` 就处于 `Killed` 形态。

#### Activity 生命周期

![Activity 生命周期](/media/android_activity_lifecycle.png)

图中列举了很多 `Activity` 的生命周期状态。其中只有三个状态是稳定的，其他状态都是过渡状态，很快就会结束。

- `Resumed`

    这个状态即前面所说的 `Active/Running` 形态。此时， `Activity` 处于 `Activity 栈` 顶，处理用户的交互。

- `Paused`

    当 `Activity` 的一部分被挡住的时候进入这个状态，这个状态下的 `Activity` 不会接收用户输入。

- `Stopped`

    当 `Activity` 完全被覆盖时进入这个状态，此时 `Activity` 不可见，仅在后台运行。

开发者不必实现所有的生命周期方法。但知道每一个生命周期状态的含义，可以让我们更好地掌控 `Activity` ，让它能更好地完成你所期望的效果。

- 启动与销毁

    在系统调用 `onCreate(Bundle)` 之后，就会马上调用 `onStart()` 。然后继续调用 `onResume()` 以进入 `Resumed` 状态，最终会停在 `Resumed` 状态，完成启动。销毁时，系统会调用 `onDestroy()` 来结束一个 Activity 的声明周期让它回到 `Killed` 形态。

  - `onCreat()`

    创建基本的 UI 元素。

  - `onPause()` 与 `onStop()`

    清除 Activity 的资源，避免浪费。

  - `onDestory()`

    因为引用会在 Activity 销毁的时候销毁，而线程不会，所以清除开启的线程。

- 暂停与恢复

    当栈顶的 Activity 部分不可见后，会导致 Activity 进入 `Pause` 形态，此时就会调用 `onPause()` 方法。当结束阻塞后，就会调用 `onResume()` 方法恢复到 `Resume` 形态。

  - `onPause()`

    释放系统资源。如 `Camera` ， `sensor` , `receivers` 。

  - `onResume()`

    需要重新初始化在 `onPause()` 中释放的资源。

- 停止

    栈顶的 `Activity` 部分不可见时，后续会有两种可能。从部分不可见到可见，也就是恢复过程。从部分不可见到完全不可见，也就是停止过程。系统在当前 Activity 不可见的时候，总会调用 `onPause()` 方法。每当 Activity 由不可见到可见时，都会调用 `onStart()` 方法。

- 重新创建

    如果 Activity 长时间处于 `Stopped` 形态而且此时系统需要更多内存或者系统内存极为紧张时，系统就会回收你的 Activity 。此时系统为了补偿你，会将 Activity 状态通过 `onSaveInstanceState(Bundle)` 方法保存到 `Bundle` 对象中，你也可以增加额外的键值对存入 `Bundle` 对象以保存这些状态。当你需要重新创建这些 Activity 的时候，保存的 `Bundle` 对象就会传递到 Activity 的 `onRestoreInstanceState(Bundle)` 方法与 `onCreate(Bundle)` 方法中。这也是 `onCreate(Bundle)` 方法中参数 `Bundle` 的来源。

    `onSaveInstanceState(Bundle)` 方法并不是每次 Activity 离开前台时都会调用。如果用户使用 `finish()` 方法结束了 Activity ，则不会调用。而且 Android 系统已经默认实现了控件的状态缓存，以此来减少开发者需要实现的缓存逻辑。

### Android 任务栈

一个 Android 应用程序功能通常会被拆分为多个 Activity ，而各个 Activity 之间通过 Intent 进行连接。而 Android 系统通过栈结构来保存整个应用的 Activity 。栈底的元素是整个任务栈的发起者。一个合理的任务调度栈不仅是性能的保证，更是提供性能的基础。

当一个应用启动时，如果当前环境中不存在该应用的任务栈，系统就会创建一个任务栈。此后，这个应用所启动的 Activity 都将在这个任务栈中被管理。这个栈也被称为一个 `Task` ，即表示若干个 Activity 的集合，他们组合在一起形成一个 `Task` 。另外，需要特别注意的是，一个 `Task` 中的 Activity 可以来自不同的应用，同一个应用的 Activity 也可能不在一个 `Task` 中。

栈结构为后进先出 (Last In First Out) 的线性表。根据 Activity 在当前栈结构中的位置，来决定该 Activity 的状态。正常情况下的 Android 任务栈，当一个 Activity 启动了另一个 Activity 的时候，新启动的 Activity 就会置于任务栈的顶端，并处于活动状态。启动它的 Activity 依然保留在任务栈中，处于停止状态，当用户按下返回键或者调用 `finish()` 方法时，系统会移除顶部 Activity ，让后面的 Activity 恢复活动状态。当然，可以通过在 `AndroidMainifest.xml` 文件中的属性 `android:launchMode` 或者是通过 `Intent` 的 `flag` 来设置特权来打破这一模式。

### Activity 启动模式

- `stardard`

    默认的启动模式，如果不指定 Activity 的启动模式，则默认使用这种方式启动 Activity 。这种启动模式每次都会创建新的 Activity 实例覆盖在原 Activity 上。

- `singleTop`

    如果指定启动 Activity 为 `singleTop` 模式，在启动时系统会判断当前栈顶 Activity 是不是要启动的 Activity 。如果是则不创建新的 Activity 而直接引用这个 Activity 。如果不是则创建新的 Activity 。这种启动模式虽然不会创建新的实例，但是系统会在 Activity 启动时调用 `onNewIntent(Intent)` 方法。

- `singleTask`

    `singleTask` 模式与 `singleTop` 模式类似。 `singleTop` 是检测栈顶元素是否是需要启动的 Activity ，而 `singleTask` 是检测整个 Activity 栈中是否存在当前需要启动的 Activity 。如果存在，则将该 Activity 置于栈顶，并将栈中该 Activity 以上的 Activity 全部销毁。如果是其他程序以 `singleTask` 模式来启动这个 Activity ，那么它将创建一个新的任务栈。需要注意的是，如果启动的模式为 `singleTask` 的 Activity 已经在一个后台任务栈中了，那么启动后，后台的任务栈将一起被切换到前台。

    这种启动模式通常可以用来退出整个应用：将主 Activity 设为 `singleTask` 模式，然后在要退出的 Activity 中转到主 Activity ，从而将主 Activity 之上的 Activity 都清除。然后重写主 Activity 的 `onNewIntent(Intent)` 方法，在方法中调用 `finish()`，将最后一个 Activity 结束掉。

- `singleInstance`

    `singleInstance` 这种启动模式和使用的浏览器工作原理类似。在多个程序中访问浏览器时，如果当前浏览器没有打开，则打开浏览器，否则会在当前打开的浏览器中访问。申明为 `singleInstance` 的 Activity 会出现在一个新的任务栈中，而且该任务栈中只存在这一个 Activity 。这种启动模式常用于需要与程序分离的界面。

关于 `singleTop` 和 `singleInstance` 这两种启动模式还有一点需要特殊说明：如果在一个 `singleTop` 或者 `singleInstance` 的 Activity 中通过 `StartActivityForResult(Intent, int)` 方法来启动另一个 Activity ，那么系统将直接返回 `Activity.RESULT_CANCELED` 而不会再去等待返回。这是由于系统在 Framework 层做了对这两种启动模式的限制。 Android 开发者认为，不同 `Task` 之间，默认是不能传递数据的。如果一定要传递，那就只能通过 `Intent` 来绑定数据。

### Intent 的 Flag

- `Intent.FLAG_ACTIVITY_NEW_TASK`

    使用一个新的 `Task` 启动一个 Activity 。该 Flag 通常使用在从 Service 中启动 Activity 的场景。由于在 Service 中并不存在 Activity 栈，所以使用该 Flag 来创建一个新的 Activity 栈，并创建新的 Activity 实例。

- `FLAG_ACTIVITY_SINGLE_TOP`

    使用 `singletop` 模式启动 Activity 。与指定 `android:launchMode="singleTop"` 效果相同。

- `FLAG_ACTIVITY_CLEAR_TOP`

    使用 `singleTask` 模式启动 Activity 。与指定 `android:launchMode="singleTask"` 效果相同。

- `FLAG_ACTIVITY_NO_HISTORY`

    使用这种模式启动 Activity ，当该 Activity 启动其他 Activity 后，该 Activity 就消失了，不会保留在 Activity 栈中。

### 清空任务栈

可以在 `AndroidMainifest.xml` 文件中的 `<activity>` 标签中使用以下几种属性来清理任务栈。

- `clearTaskOnLaunch`

    `clearTaskOnLaunch` 属性就是在每次返回该 Activity 时，都将该 Activity 之上的所有 Activity 都清除。通过这个属性，可以让这个 `Task` 每次在初始化的时候，都只有这一个 Activity 。

- `finishOnTaskLaunch`

    `finishOnTaskLaunch` 属性与 `clearTaskOnLaunch` 属性类似，只不过 `clearTaskOnLaunch` 作用在别人身上，而 `finishOnTaskLaunch` 作用在自己身上。通过这个属性，当离开这个 Activity 所处的 Task ，用户再返回时，该 Activity 就会被结束掉。

- `alwaysRetainTaskState`

    如果将Activity的这个属性设置为 `true` ，那么该 Activity 所在的 `Task` 将不接受任何清理命令，一直保持当前 `Task` 状态。

### 任务栈使用

我们使用 Activity 任务栈的各种启动模式和清理方法，是为了更好地使用 App 中的 Activity 。合理地设置 Activity 的启动模式会让程序运行更有效率，用户体验更好。但任务栈虽好，却也不能滥用。如果过度地使用 Activity 任务栈，则会导致整个 App 的栈管理混乱。不仅不利于以后程序的拓展，而且在容易出现由于任务栈导致的显示异常。这样的 Bug 是很难调试的。所以，在 App 中使用 Activity 任务栈一定要根据实际项目的需要，而不是为了使用任务栈而使用任务栈。

[↑ 目录](#index)

------

<h2 id="chap9">系统信息与安全机制</h2>

### 获取系统信息

要获取系统的配置信息，通常可以从以下两个方面获取。

- `android.os.Build`

    该类里面的信息非常丰富，它包含了系统编译时的大量设备、配置信息。

  - Build.BOARD // 主板
  - Build.BRAND // Android 系统定制商
  - Build.SUPPORTED_ABIS // CPU 指令集
  - Build.DEVICE // 设备参数
  - Build.DISPLAY // 显示屏参数
  - Build.FINGERPRINT // 唯一编号
  - Build.SERIAL // 硬件序列号
  - Build.ID // 修订版本列表
  - Build.MANUFACTURER // 硬件制造商
  - Build.MODEL // 版本
  - Build.HARDWARE // 硬件名
  - Build.PRODUCT // 手机产品名
  - Build.TAGS // 描述 Build 的标签
  - Build.TYPE // Builder 类型
  - Build.VERSION.CODENAME // 当前开发代号
  - Build.VERSION.INCREMENTAL // 源码控制版本号
  - Build.VERSION.RELEASE // 版本字符串
  - Build.VERSION.SDK_INT // 版本号
  - Build.HOST // Host值
  - Build.USER // User名
  - Build.TIME // 编译时间

- `SystemProperty`

    `SystemProperty` 包含许多系统配置属性值和参数，通过 `System.getProperty(String)` 可以访问到系统的属性值。很多信息与通过 `android.os.Build` 获取的值是相同的。

  - os.version // OS 版本
  - os.name // OS 名称
  - os.arch // OS 架构
  - user.home // Home 属性
  - user.name // Name 属性
  - user.dir // Dir 属性
  - user.timezone // 时区
  - path.separator // 路径分隔符
  - line.separator // 行分隔符
  - file.separator // 文件分隔符
  - java.vendor.url // Java vender URL 属性
  - java.class.path // Java Class 路径
  - java.class.version // Java Class 版本
  - java.vendor // Java Vender 属性
  - java.version // Java 版本
  - java.home // Java Home 属性

这些系统信息最的来源是系统的 `system/build.prop` 文件。

### `PackageManager`

Android 系统提供 `PackageManager` 负责管理所有已安装的应用。

`PackageManager` 可以通过调用各种方法，返回不同类型的对象。 `PackageManager` 经常使用以下方法。

- `getPackageManager()`

    返回一个 `PackageManager` 对象。

- `getApplicationInfo(String, int)`

    以 `ApplicationInfo` 的形式返回指定包名的 ApplicationInfo 。第一个参数为包名，第二个参数为 Flag 。

- `getApplicationIcon(String)`

    返回指定包名的 Icon 。参数为包名。

- `getInstalledApplications(int)`

    以 `ApplicationInfo` 的形式返回安装的应用。参数为 Flag 。

- `getInstalledPackages(int)`

    以 `PackageInfo` 的形式返回安装的应用。参数为 Flag 。

- `queryIntentActivities(Intent, int)`

    返回指定 `intent` 的 `ResolveInfo` 对象、 `Activity` 集合。第一个参数为 `Intent` 对象，第二个参数为 Flag 。

- `queryIntentServices(Intent, int)`

    返回指定 `intent` 的 `ResolveInfo` 对象、 `Service` 集合。第一个参数为 `Intent` 对象，第二个参数为 Flag 。

- `resolveActivity(Intent, int)`

    返回指定 `intent` 的 `Activity` 。第一个参数为 `Intent` 对象，第二个参数为 Flag 。

- `resolveService(Intent, int)`

    返回指定 `intent` 的 `Service` 。第一个参数为 `Intent` 对象，第二个参数为 Flag 。

下面列举了一些常用的系统封装信息。

- `ActivityInfo`

    `ActivityInfo` 封装了在 `AndroidMainifest.xml` 文件中 `<activity>` 和 `<receiver>` 之间的所有信息，包括 `name` ， `icon` ， `label` ， `launchmod` 等。

- `ServiceInfo`

    `ServiceInfo` 封装了 `<service>` 之间的所有信息。

- `ApplicationInfo`

    `ApplicationInfo` 封装了 `<application>` 之间的信息。特别的是， `ApplicationInfo` 包含很多 `Flag` ，`FLAG_SYSTEM` 表示为系统应用， `FLAG_UPDATED_SYSTEM_APP` 表示为经过升级的系统应用， `FLAG_EXTERNAL_STORAGE` 表示为安装在 SDCard 上的应用等。通过这些 `Flag` 可以很方便地判断应用的类型。

- `PackageInfo`

    `PackageInfo` 用于封装 `AndroidMainifest.xml` 文件的所有 `Activity` ， `Service` 等信息。

- `ResolveInfo`

    `ResolveInfo` 封装的是包含 `<intent>` 信息的上一级信息，所以它可以返回 `ActivityInfo` ， `ServiceInfo` 等包含 `<intent>` 的信息，它经常用来帮助我们找到那些包含特定 Intent 条件的信息，如带分享功能、播放功能的应用。

### `ActivityManager`

`PackageManager` 重点在于获得应用的包信息，而 `ActivityManager` 重点在与获得在运行的应用程序信息。 `ActivityManager` 封装了不少对象。

- `ActivityManager.MemoryInfo`

    `ActivityManager.MemoryInfo` 有几个非常重要的字段

  - `availMem` 系统可用内存
  - `totalMem` 总内存
  - `threshold` 低内存的阈值，即区分是否低内存的临界值
  - `lowMemory` 是否处于低内存。

- `Debug.MemoryInfo`

    `ActivityManager.MemoryInfo` 通常用于获取全局的内存使用信息，而 `Debug.MemoryInfo` 用于统计进程下的内存信息。

- `RunningAppProcessInfo`

    `RunningAppProcessInfo` 是运行进程的信息，存储的字段是进程相关的信息。

  - `processName` 进程名
  - `pid` 进程 pid
  - `uid` 进程 uid
  - `pkgList` 该进程下的所有包

- `RunningServiceInfo`

    `RunningServiceInfo` 用于封装运行的服务信息，在它里面同样包含了一些服务进程的信息，同时还有一些其他信息。

  - `activeSince` 第一次被激活的时间、方式
  - `foreground` 服务是否在后台执行

### 解析 `Packages.xml`

`packages.xml` 文件位于 `/data/system/` 目录下。在系统初始化的时候， `PackageManager` 的底层实现类 `PackageManagerService` 会扫描系统中的一些特定的目录，并解析其中的 Apk 文件。同时把它获得的应用信息，保存在 `packages.xml` 文件中。当系统中的 Apk 安装、删除、升级时，它也会被更新。

`packages.xml` 文件非常复杂，先了解一下这个文件所包含的信息点标签。

- `<permissions>` 标签

    `<permissions>` 标签定义了目前系统中的所有权限，并分为两类：系统定义的( `package` 属性为 `Android` )和 Apk 定义的( `package` 属性为 Apk 包名)。

- `<package>` 标签

    `<package>` 标签代表了一个 Apk 的属性。其中各节点信息的含义如下所示。

  - `name` Apk 的包名。
  - `codePath` Apk 安装路径。主要有 `/system/app` 和 `/data/app` 两种。 `/system/app` 存放系统级别的 Apk 或者是厂商定制的 Apk ， `/data/app` 存放用户安装的第三方 Apk 。
  - `userId` 用户 ID
  - `version` 版本号

- `<perms>` 标签

    对应 Apk 的 `AndroidManifest.xml` 文件中的 `<uses-permission>` 标签，记录 Apk 的权限信息。

### Android 安全机制

#### 安全机制

- 代码混淆 proguard

    混淆关键代码、替换命名让破坏者阅读困难，同时也可以压缩代码、优化编译后的 Java 字节码。

- `AndroidMainifest.xml` 文件权限声明、权限检查机制

    任何应用程序在使用 Android 受限资源的时候，都需要显示向系统声明所需要的权限。只有当一个应用具有相应的权限，才能在申请受限资源的时候，通过权限机制的检查并使用系统的 `Binder` 对象完成对系统服务的调用。

    Android 系统通常按照以下顺序来检查操作者的权限。首先，判断 permission 名称。如果为空则直接返回 `PERMISSION_DENIED` 。其次，判断 Uid 。如果为 `0` 则为Root权限，不做权限控制；如果为 `System Server` 的 Uid 则为系统服务，不做权限控制；如果 Uid 与参数中的请求 Uid 不同，则返回 `PERMISSION_DENIED` 。最后，通过调用 `PackageManagerService.checkUidPermission(String, int)` 方法判断该 Uid 是否具有相应的权限。该方法会去 XML 的权限列表和系统级的 `platform.xml` 中进行查找。

    但是这种机制也有先天性不足，如以下几项。

  - 被授予的权限无法停止。
  - 在应用声明 App 使用权限时，用户无法针对部分权限进行限制。
  - 权限的声明机制与用户的安全理念相关。

- 数字证书

    Android 中所有的 App 都会有一个数字证书，这就是 App 的签名。数字证书用于保护 App 的作者对其 App 的信任关系，只有拥有相同数字签名的 App ，才会在升级时被认为是同一 App 。而且 Android 系统不会安装没有签名的 App 。

- Uid 、访问权限控制

    Android 继承了 Linux 的安全特性。通常情况下，只有 System 、 root 用户才有权限访问到系统文件，而一般用户无法访问。

- 沙箱隔离

    Android 的 App 运行在虚拟机中，因此才有沙箱机制，可以让应用之间相互隔离。通常情况下，不同的应用之间不能互相访问，每个 App 都有与之对应的 Uid ，每个 App 也运行在单独的虚拟机中，与其他应用完全隔离。在实现安全机制的基础上，也让应用之间能够互不影响，即使一个应用崩溃，也不会导致其他应用异常。

#### 安全隐患

- 代码漏洞

    这个问题存在于世界上所有的程序中，没有谁敢保证自己的程序永远不会有 Bug 。遇到这种问题，只能尽快升级系统 OS 版本、更新补丁，才能杜绝利用漏洞的攻击者。

- Root 风险

    Root 权限是指 Android 的系统管理员权限，具有 Root 权限的用户可以访问和修改手机中几乎所有的文件。 Root 后的手机少了 Linux 的天然屏障，整个系统核心就完全暴露在入侵者面前，在没有察觉的情况下大肆破坏。针对普通用户，希望尽量不要 Root 手机以免带来不必要的损失。

- 安全机制不健全

    由于 Android 的权限管理机制并不完美。所以很多手机开发商，通常会在 ROM 中增加自己的一套权限管理工具来帮助用户控制手机中应用的权限。

- 用户安全意识

    用户可以通过在正规应用市场下载应用、并在安装应用时通过列出来的应用权限申请信息来大致判断一个应用的安全性。

- Android 开发原则与安全

    Google 本着开源的精神开放了 Android 的源代码，但随之而来的各种安全问题也让 Android 饱受诟病。过度的开放与可定制化，不仅造成了 Android 的碎片化严重，同时也给很多不法应用以可乘之机。

#### APK 反编译

***TODO***

#### APK 加密

使用 ProGuard 对 Apk 进行混淆处理，用无意义的字母来重命名类、字段、方法和属性。还可以删除无用的类、字段、方法和属性，以及删除没用的注释，最大限度地优化字节码文件。ProGuard 的配置在模块的 `build.gradle` 文件中。 `minifyEnabled` 属性就是控制是否启用 ProGuard 的开关。

```groovy
buildTypes {
    release {
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

[↑ 目录](#index)

------

<h2 id="chap10">Android 性能优化</h2>

### 布局优化

在 Android 中，系统通过 `VSYNC` 信号触发对 UI 的渲染、重绘，其间隔时间是 16ms 。如果不能在16ms内完成绘制，就会造成当前应当重绘的帧被未完成的逻辑阻塞。该帧会被丢弃，等待下次信号再继续绘制，导致 16*2ms 内都显示同一帧画面，这就是画面卡顿的原因。

Android 系统提供了检测 UI 渲染时间的工具。打开 `开发者选项` 选择 `Profile GPU Rendering` ，并选择 `On screen as bars` 。这时候在屏幕上将显示一些条形图。每一条柱状线都包含三部分，蓝色代表测量绘制 Display List 的时间，红色代表 OpenGL 渲染 Display List 所需要的时间，黄色代表 CPU 等待 GPU 处理的时间。中间的绿色横线代表 VSYNC 时间 16ms ，需要尽量将所有条形图都控制在这条绿线之下。

过度绘制会浪费很多 CPU 、 GPU 资源。 Android 开发者选项中提供了 `Debug GPU Overdraw` 工具。激活后，可以通过界面上的颜色来判断 Overdraw 的次数。尽量增大蓝色的区域、减少红色区域。

在 Android 中，系统对 View 进行测量、布局和绘制时，是通过对 View 树的遍历来进行操作的。如果一个 View 树的高度太高，就会严重影响测量、布局和绘制的速度。 Google 建议 View 树的高度不宜超过10层。在布局时，需要根据自身布局的特点来选择不同的 Layout 组件，从而避免通过某一种 Layout 组件来实现功能时的局限性造成嵌套过多的情况发生。

#### `<include>` 标签

可以使用 `<include>` 标签来定义一些共通 UI 。在使用该共通UI的布局文件中通过 `<include>` 标签的 `layout` 属性添加对这个共通 UI 的 ID 的引用。

#### `<ViewStub>` 延迟加载

`<ViewStub>` 是一个非常轻量级的组件，它不仅不可视，而且大小为 0 。在布局中添加 `<ViewStub>` 节点并使用 `layout` 属性引用待加载的布局。要重新加载待显示的布局，可以通过 `findViewById(int)` 方法找到 `<ViewStub>` 组件，然后通过调用 `ViewStub` 的 `setVisibility(VISIBLE)` 或调用 `ViewStub` 的 `inflate()` 方法来显示这个 View 。这两种方式都可以让 ViewStub 重新展开，显示引用的布局。唯一的区别就是 `inflate()` 方法可以返回引用的布局，从而可以再通过 `View.findViewById(int)` 方法找到对应的控件。一旦 `<ViewStub>` 被设置为可见或是被 inflate 了， `<ViewStub>` 就不存在了，取而代之的是被 inflate 的 Layout ，并将这个 Layout 的 ID 重新设置为 `<ViewStub>` 中通过 `android:inflatedId` 属性所指定的 ID 。

`<ViewStub>` 标签与设置 `View.GONE` 这种方式来隐藏一个 View 的区别是， `<ViewStub>` 标签只会在显示时，才去渲染整个布局，而 `View.GONE` ，在初始化布局树的时候就已经添加在布局树上了，相比之下 `<ViewStub>` 标签的布局具有更高的效率。

#### Hierarchy Viewer

[ViewServer](https://github.com/romainguy/ViewServer)

该工具位于 `sdk\tools` 目录下，在命令行中输入 `hierarchyviewer.bat` 启动程序。选择要调试的进程 → 点击 `Load View Hierarchy` → 点击 `Profile Node` 重新进行计算，获取绘制信息。

### 内存优化

通常内存是指手机的 RAM ，包括以下几个部分。

- 寄存器 `Registers`

    速度最快的存储场所，因为寄存器位于处理器内部。在程序中无法控制。

- 栈 `Stack`

    存放基本类型的数据和对象的引用，但对象本身不存放在栈中，而是存放在堆中。

- 堆 `Heap`

    堆内存用来存放由 `new` 创建的对象和数组。在堆中分配的内存，由 Java 虚拟机的自动垃圾回收器（GC）管理。

    在程序中，可以使用如下所示的代码来获得堆的大小，所谓的内存分析，正是分析 Heap 中的内存状态。

    ```java
    ActivityManager manager = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
    int heapSize = manager.getLargeMemoryClass();
    ```

- 静态存储区域 `Static Field`

    静态存储区域就是指在固定的位置存放应用程序运行时一直存在的数据， Java 在内存中专门划分了一个静态存储区域来管理一些特殊的数据变量如静态的数据变量。

- 常量池 `Constant Pool`

    JVM 虚拟机必须为每个被装载的类型维护一个常量池。常量池就是该类型所用到常量的一个有序集合，包括直接常量（基本类型，String）和对其他类型、字段和方法的符号引用。

Java 创建了垃圾收集器线程（Garbage Collection Thread）来自动进行资源的管理。这样做的好处是大大降低了程序开发人员对内存管理的繁琐工作。但这也带来了很多问题，例如 Java 的 GC 是系统自动进行的，但何时进行却是开发者无法控制的，即使调用 `System.gc()` 方法，也只是建议系统进行 GC ，但系统是否采纳你的建议，那就不一定了。 JVM 虚拟机虽然能够自动控制 GC ，但是再强大的算法，也难免会存在部分对象忘记回收的现象发生，这就是造成内存泄漏的原因。

#### `Bitmap` 优化

- 使用适当分辨率和大小的图片

    由于 Android 系统在做资源适配的时候会对不同分辨率文件夹下的图片进行缩放来适配相应的分辨率，如果图片分辨率与资源文件夹分辨率不匹配或者图片分辨率太高，就会导致系统消耗更多的内存资源。同时，在适当的时候，应该显示合适大小的图片，例如在图片列表界面可以使用图片的缩略图thumbnails，而在显示详细图片的时候再显示原图；或者在对图像要求不高的地方，尽量降低图片的精度。

- 及时回收内存

    一旦使用完 Bitmap 后，一定要及时使用 `bitmap.recycle()` 方法释放内存资源。自 Android 3.0 之后，由于 Bitmap 被放置到了堆中，其内存由 GC 管理，就不需要进行释放了。

- 使用图片缓存

    通过内存缓存（LruCache）和硬盘缓存（DiskLruCache）可以更好地使用 Bitmap 。

#### 代码优化

- 对常量使用 `static` 修饰符。
- 使用静态方法，静态方法会比普通方法提高 15% 左右的访问速度。
- 减少不必要的成员变量，这点在 Android Lint 工具上已经集成检测了。如果一个变量可以定义为局部变量，则会建议你不要定义为成员变量。
- 减少不必要的对象。使用基础类型会比使用对象更加节省资源，同时更应该避免频繁创建短作用域的变量。
- 尽量不要使用枚举、少用迭代器。
- 对 `Cursor` 、 `Receiver` 、 `Sensor` 、 `File` 等对象，要非常注意对它们的创建、回收与注册、解注册。
- 避免使用 IOC 框架。 IOC 通常使用注解、反射来进行实现，虽然现在 Java 对反射的效率已经进行了很好的优化，但大量使用反射依然会带来性能的下降。
- 使用 RenderScript 、 OpenGL 来进行非常复杂的绘图操作。
- 使用 `SurfaceView` 来替代 `View` 进行大量、频繁的绘图操作。
- 尽量使用视图缓存，而不是每次都执行 `inflate()` 方法解析视图。

### Lint

Android Lint 工具是 Android Studio 中集成的一个 Android 代码提示工具，它可以给你的布局、代码提供非常强大的帮助。

### Memory Monitor

Memory Monitor 是 Android Studio 自带的内存监视工具，它可以很好地帮助我们进行内存实时分析。通过点击 Android Studio 右下角的 `Memory Monitor` 标签，打开工具。

### TraceView

TraceView 是一个 Android 下的可视化性能调查工具，它用来分析 TraceView 日志。

- 生成 TraceView 日志

  - 通过代码生成精确范围的 TraceView 日志

    通过调用 `Debug.startMethodTracing()` 方法开启监听，通过调用 `Debug.stopMethodTracing()` 方法结束监听。我们可以使用这两个方法来包围要监听的代码块。 TraceView 的日志将会保存到 `/sdcard/dmtrace.trace` 目录下，因此，需要在 Mainifest 文件中增加如下所示的权限。

    ```xml
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ```

  - 通过 Android Device Monitor 生成 TraceView 日志

    打开 Android Studio 的 Android Device Monitor 工具，选择要调试的进程，点击工具栏中的 `start method profiling` 按钮。弹出提示，选择监听的模式。 TraceView 提供了两种监听方式。

    - 整体监听

        跟踪每个方法执行的全部过程，这种方式资源消耗较大。

    - 抽样监听

        按照指定的频率来进行抽样调查，这种方式需要执行较长时间来获取较准确的样本数据。

    执行一段时间后，再次点击 `start method profiling` 按钮即可结束监听。

- 打开 TraceView 日志

    导出的 TraceView 日志文件可以使用 SDK 中的 `sdk\tools\traceview.bat` 工具打开。或者在 Android Device Monitor 工具中的 `File` 菜单下选择 `Open File…` 选项打开 TraceView 日志文件。

- 分析 TraceView 日志

    TraceView 的分析界面分为两部分，上面是用于显示方法执行时间的时间轴区域，下面是显示详细信息的 profile 区域。时间轴区域显示了不同线程在不同的时间段内的执行情况，在时间轴中，每一行都代表了一个独立的线程。使用鼠标滚轮可以放大时间轴。不同的色块代表了不同的执行方法，色块的长度，代表了方法所执行的时间。 profile 区域内显示了你选择的色块所代表的方法在该色块所处的时间段内的性能分析。在 profile 区域中主要显示以下的信息。

    - Incl CPU Time —— 某方法占用CPU的时间。
    - Excl CPU Time —— 某方法本身（不包括子方法）占用CPU的时间。
    - Incl Real Time —— 某方法真正执行的时间。
    - Excl Real Time —— 某方法本身（不包括子方法）真正执行的时间。
    - Calls + RecurCalls —— 调用次数+递归回调的次数。

### Memory Analyzer Tool

Memory Analyzer Tool 工具是一个分析内存的强力助手。

- 生成 HPROF 文件

    打开 Android Studio 的 Android Device Monitor 工具，选择要监听的线程，并点击菜单栏中的 `Update Heap` 按钮。在 `Heap` 标签中，点击 `Cause GC` 按钮，就会显示出当前的内存状态。点击菜单栏的 `Dump HPROF File` 按钮，等待几秒钟后，系统就会生成一个 `.hprof` 文件。不过这个文件还不能直接使用 MAT 工具进行分析，需要进行格式转换。在命令行下，切换到 SDK 的 `platform-tools` 目录下，使用 `hprof-conv` 工具帮助我们进行转换，命令格式为 `hprof-conv infile outfile` 。生成的文件就可以进行内存分析了。

- 分析 HPROF 文件

    打开 MAT 工具，选择 `Open a Heap Dump` 选项。等待文件导入后，显示分析结果。 MAT 功能强大，这里主要看以下几个功能。

  - Histogram

    Histogram 直方图，用于显示内存中每个对象的数量、大小和名称。打开 `Histogram` 标签，在选择的对象上单击鼠标右键，在弹出的快捷菜单中选择 `List objects-with incoming references` 选项查看具体的对象。

  - Dominator Tree

    Dominator Tree 支配树会将内存中的对象按照大小进行排序，并显示对象之间的引用结构。打开 `Dominator Tree` 标签，通过分析内存占用大的对象来找出内存消耗的原因。

### `Dumpsys` 命令

使用 `Dumpsys` 命令可以列出 Android 系统相关的信息和服务状态。

[Dumpsys的官方信息](https://source.android.com/devices/input/diagnostics?hl=zh-cn)

常用的 `Dumpsys` 参数：

| 命令       | 功能                      |
| --------- | ------------------------- |
| activity  | 显示所有的 Activity 栈的信息 |
| meminfo   | 内存信息                   |
| battery   | 电池信息                   |
| package   | 包信息                     |
| wifi      | 显示 Wi-Fi 信息            |
| alarm     | 显示 alarm 信息            |
| procstats | 显示内存状态                |

[↑ 目录](#index)

------

<h2 id="chap11">搭建云端服务器</h2>

***TODO***

[↑ 目录](#index)

------

<h2 id="chap12">Android Lollipop 新特性详解</h2>

### Material Design

### MD 主题

### Palette

### 视图

### 阴影

### Tinting

### Clipping

### 列表

### 卡片

### Activity 过渡动画

### MD 动画效果

### Toolbar

### Notification

[↑ 目录](#index)

------

<h2 id="chap13">Android 实例提高</h2>

***TODO***

[↑ 目录](#index)

------
