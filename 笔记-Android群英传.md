# Android 群英传

<h2 id="index">目录</h2>

- [第一章 Android 体系与系统架构](#chap1)
- [第二章 Android 开发工具](#chap2)
- [第三章 Android 控件](#chap3)
- [第四章 ListView 使用](#chap4)
- [第五章 Android Scroll 分析](#chap5)
- [第六章 Android 绘图机制](#chap6)
- [第七章 Android 动画机制](#chap7)

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

    **例：**让子 View 跟随手指滑动。但是在手指离开屏幕时，让子 View 平滑的移动到初始位置。

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

    **例：**实现类似 QQ 滑动侧边栏的布局。初始时显示内容界面。当用户手指滑动超过一段距离时，内容界面侧滑显示菜单界面。

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

**例：**创建一个仪表盘

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

**例：**绘制一个正弦曲线。使用一个 `Path` 对象保存正弦函数上的坐标点，在子线程的 `while` 循环中，不断改变横纵坐标值。

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

**例：**实现一个简单的绘图板。在 `SurfaceView#onTouchEvent(MotionEvent)` 中记录 `Path` 路径，通过 `Path` 对象进行绘图。并在 `draw()` 方法中进行绘制。绘制不需要很频繁。因此我们可以在子线程中进行 `sleep` 操作，尽可能地节省系统资源。判断 `draw()` 方法所用逻辑时长确定 `sleep` 的时长， `sleep` 时间的取值一般在 50ms 到 100ms 左右。

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
    long start = System.currentTimeMillis();

    while (mIsDrawing) {
        draw();
    }

    long end = System.currentTimeMillis();

    if (end - start < 100) {
        try {
            Thread.sleep(100 - (end - start));

        } catch (InterruptedException e) {
            e.printStackTrace();
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

### View 动画

### 属性动画

### 布局动画

### 插值器 `Interpolators`

### 自定义动画

### SVG 矢量动画

### 动画示例

[↑ 目录](#index)

------