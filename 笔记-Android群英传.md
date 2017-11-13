# Android 群英传

<h2 id="index">目录</h2>

- [Android 体系与系统架构](#chap1)
- [Android 开发工具](#chap2)
- [Android 控件](#chap3)

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

例：拓展 TextView 创建新的控件。使其背景更加丰富，多绘制几层背景。

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

例：拓展 TextView 创建新的控件。利用 `LinearGradient Shader` 和 `Matrix` 实现动态文字闪动效果。

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

例：自定义组合 TopBar 。

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

### 自定义 ViewGroup

### 事件拦截机制

[↑ 目录](#index)

------