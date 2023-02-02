---

title: Compose 使用入门

toc: true

date: 2023-02-01 21:18:22

author: <a href="https://https://github.com/MemoryLimitExceeded">MemoryLimitExceeded</a>

revision: <a href="https://github.com/ALightGroup">ALG</a>

tags: 

  - Compose
  - android

categories: Compose

---

## 概览

Compose 是一种全新构建 Android 原生界面的组件，抛弃了传统的 xml 声明方式，直接在 kt 文件中声明调用构建界面。

## 集成

在模块 build.gradle 文件中添加

```groovy
android {
    buildFeatures {
        compose true
    }
    composeOptions {
        kotlinCompilerExtensionVersion = "1.3.2"
    }
}
```

代码块 composeOptions 中定义的 Kotlin 编译器扩展版本需要与使用的 Kotlin 版本对应，对应关系表查看[兼容性对应图](https://developer.android.google.cn/jetpack/androidx/releases/compose-kotlin)

之后添加以下依赖

```groovy
dependencies {

    // 必选项
    // Compose 编程模型和状态管理的基本构建块，以及 Compose Compiler 插件针对的核心运行时。
    implementation "androidx.compose.runtime:runtime"
    // 借助 Kotlin 编译器插件，转换 @Composable functions（可组合函数）并启用优化功能。
    implementation "androidx.compose.compiler:compiler"
    // 与设备互动所需的 Compose UI 的基本组件，包括布局、绘图和输入。
    implementation "androidx.compose.ui:ui"
    // 使用 Material Design 组件构建 Jetpack Compose 界面。
    implementation "androidx.compose.material:material"
    // 使用现成可用的构建块编写 Jetpack Compose 应用，还可扩展 Foundation 以构建您自己的设计系统元素。
    implementation "androidx.compose.foundation:foundation"

    // 可选项
    // Android Studio 预览支持
    implementation "androidx.compose.ui:ui-tooling-preview"
    implementation "androidx.compose.ui:ui-tooling"
    // compose Activity
    implementation "androidx.activity:activity-compose"
    // compose ViewModels
    implementation "androidx.lifecycle:lifecycle-viewmodel-compose"
    // compose LiveData
    implementation "androidx.compose.runtime:runtime-livedata"
    // compose constraintlayout
    implementation "androidx.constraintlayout:constraintlayout-compose"
    // compose Navigation
    implementation "androidx.navigation:navigation-compose"
    // compose glide
    implementation "com.github.bumptech.glide:compose"

}
```

可通过引入 BoM 库进行依赖的版本管理，在进行依赖时仅需关注 BoM 库版本即可。

```groovy
dependencies {
    // 引入 BoM 库
    implementation platform('androidx.compose:compose-bom:2022.12.00')

    // 将 BoM 库中的该依赖版本替换为 1.1.0-alpha01
    implementation 'androidx.compose.material3:material3:1.1.0-alpha01'

    // 添加 BoM 库中包含的依赖
    implementation 'androidx.compose.foundation:foundation'
}
```

如需了解哪些 Compose 库版本已映射到特定 BoM 版本，请查看 [BoM 到库的版本映射](https://developer.android.google.cn/jetpack/compose/setup#bom-version-mapping)。

## 引入使用

### 1.androidx.activity.ComponentActivity

通常情况下使用可选依赖的 compose Activity 中，ComponentActivity.setContent() 会做为 ComposeView 的入口

```kotlin
// androidx.activity.compose.ComponentActivity.kt
public fun ComponentActivity.setContent(
    parent: CompositionContext? = null,
    content: @Composable () -> Unit
)
// MainActivity.kt
class MainActivity : androidx.activity.ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            // 你的界面
        }
    }

}
```

### 2.ComposeView 做为 View

```kotlin
// ComposeViewFragment.kt
class ComposeViewFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        return ComposeView(requireContext()).apply {
            setContent {
                // 你的界面
            }
        }
    }

}
// XmlFragment.kt
class XmlFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        val rootView = LayoutInflater.from(context).inflate(R.layout_fragment_xml, container, false)
        val composeView = rootView.findViewById<ComposeView>(R.id.compose_view)
        composeView.setContent {
            // 你的界面
        }
        return rootView
    }

}
```

> 这种方式在部分场景下会报 java.lang.IllegalStateException: ViewTreeLifecycleOwner not found from androidx.compose.ui.platform.ComposeView，比如 dialog.setContentView(ComposeView(Context))
>
> 使用过程中请注意。

## 特性

Jetpack Compose 在执行可组合项构建界面时被称之为组合，组合又分为**初始组合**和**重组**。

**初始组合**：首次运行可组合项的组合。

**重组**：因数据源发生变化而进行的组合。

默认情况下的 Jetpack Compose 仅会进行一次组合，因为此时的数据源对于它来说是常量，即便是在外部修改数据源也不会发生重组进行界面更新。

从而就需要引入 mutableStateOf 来修饰数据源告诉 Jetpack Compose 这是一个变化的数据，之后每当数据源变更时就会使得依赖该数据源的可组合项进行重组。

```kotlin
@Composable
fun MyText() {
    var text = mutableStateOf("哈哈哈")
    Text(
        text = text,
        modifier = Modifier.clickable {
            text = "呵呵"
        }
    )
}
```

![](https://raw.githubusercontent.com/ALightGroup/ALightGroup.github.io/alg-img/7be6fa6f-3272-4bcf-b10d-aa971950d969.gif)

不过这种方式还不能实现数据源更新界面也随之更新，这是因为进行重组的时候 text 仍会被 "哈哈哈" 所覆盖，还需要加上 remember API 将对象进行存储。

```kotlin
@Composable
fun MyText() {
    var text = remember {mutableStateOf("哈哈哈")}
    Text(
        text = text,
        modifier = Modifier.clickable {
            text = "呵呵"
        }
    )
}
```

由 remember 修饰时里面的数据在进行重组的时候就不会被重新初始化，从而能够保证更新的数据能够正确展示到界面中。

![](https://raw.githubusercontent.com/ALightGroup/ALightGroup.github.io/alg-img/c6cb991b-c833-45b5-81e5-5f0f9de837d0.gif)

## 基本组件

Compose 的基本组件可分为布局和控件，常用布局有：

1. Box
2. Row / Column
3. Constraintlayout (需要依赖 compose constraintlayout)
4. LazyRow / LazyColumn

常用控件有：

1. Text
2. TextField
3. Image
4. Dialog

### 常用布局

#### Box

可看做是 Compose 版的 FrameLayout

```kotlin
// Box.kt
@Composable
inline fun Box(
    modifier: Modifier = Modifier,
    contentAlignment: Alignment = Alignment.TopStart,
    propagateMinConstraints: Boolean = false,
    content: @Composable BoxScope.() -> Unit
)

// 使用
Box{
    // 你的内容
}
```

modifier：修饰符

contentAlignment：内容对齐方法，等价于 android:gravity

propagateMinConstraints：是否将传入的最小值约束传递给内容。

content：Box 中的内容

#### Row / Column

两个可分别看做 LinearLayout 的水平排列和垂直排列

```kotlin
// Row.kt
@Composable
inline fun Row(
    modifier: Modifier = Modifier,
    horizontalArrangement: Arrangement.Horizontal = Arrangement.Start,
    verticalAlignment: Alignment.Vertical = Alignment.Top,
    content: @Composable RowScope.() -> Unit
)
// Column.kt
@Composable
inline fun Column(
    modifier: Modifier = Modifier,
    verticalArrangement: Arrangement.Vertical = Arrangement.Top,
    horizontalAlignment: Alignment.Horizontal = Alignment.Start,
    content: @Composable ColumnScope.() -> Unit
)
// 使用
Row{
    // 你的内容
}
Column{
    // 你的内容
}
```

modifier：修饰符

horizontalArrangement/verticalArrangement：水平/垂直排列方式

verticalAlignment/horizontalAlignment：内容垂直/水平对齐方式，等价于垂直/水平方向上的 android:gravity

content：内容

#### Constraintlayout

Compose 版的约束布局

```kotlin
// Constraintlayout.kt
@Composable
inline fun ConstraintLayout(
    modifier: Modifier = Modifier,
    optimizationLevel: Int = Optimizer.OPTIMIZATION_STANDARD,
    crossinline content: @Composable ConstraintLayoutScope.() -> Unit
)
```

modifier：修饰符

optimizationLevel：（作用未知）

content：内容

```kotlin
// 使用
ConstraintLayout {
    val (title, content) = createRefs()
    Text(text = "",
        modifier = Modifier
            .constrainAs(title) {
                top.linkTo(parent.top)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
                // 在满足约束条件下宽/高尽可能的大
                width = Dimension.fillToConstraints
                height = Dimension.fillToConstraints
            })
    Text(text = "",
        modifier = Modifier
            .constrainAs(content) {
                top.linkTo(title.bottom)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
                // 在满足约束条件下宽/高尽可能的大
                width = Dimension.fillToConstraints
                height = Dimension.fillToConstraints
            })
}
```

首先调用 createRefs() 创建需要进行约束控件的引用，之后通过 Modifier.constrainAs() 将当前控件与引用进行绑定，之后在方法后面的 lambda 中采用类似 xml 中使用约束布局的方法进行约束。

> [Compose 中的 ConstraintLayout](https://developer.android.google.cn/jetpack/compose/layouts/constraintlayout?hl=zh-cn)

#### LazyRow / LazyColumn

两个可分别看做 RecyclerView 的水平排列和垂直排列

```kotlin
// LazyDsl.kt
@Composable
fun LazyRow(
    modifier: Modifier = Modifier,
    state: LazyListState = rememberLazyListState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    horizontalArrangement: Arrangement.Horizontal =
        if (!reverseLayout) Arrangement.Start else Arrangement.End,
    verticalAlignment: Alignment.Vertical = Alignment.Top,
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    userScrollEnabled: Boolean = true,
    content: LazyListScope.() -> Unit
)
@Composable
fun LazyColumn(
    modifier: Modifier = Modifier,
    state: LazyListState = rememberLazyListState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    verticalArrangement: Arrangement.Vertical =
        if (!reverseLayout) Arrangement.Top else Arrangement.Bottom,
    horizontalAlignment: Alignment.Horizontal = Alignment.Start,
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    userScrollEnabled: Boolean = true,
    content: LazyListScope.() -> Unit
)
```

modifier：修饰符

state：用于控制或观察列表状态的对象

contentPadding：内边距

reverseLayout：是否反向排列列表

horizontalArrangement/verticalArrangement：水平/垂直排列方式

verticalAlignment/horizontalAlignment：内容垂直/水平对齐方式，等价于垂直/水平方向上的 android:gravity

flingBehavior：惯性滑动行为

userScrollEnabled：是否允许进行触摸滑动

content：内容

```kotlin
// 使用
LazyRow {
    item {
        // item 1
    }
    item {
        // item 2
    }
    // 添加一组 item
    list.forEach {
        item {
            // listItem
        }
    }
    // 添加一组 item 的另一种方法
    items(list) { data ->
        // listItem
    }
}
LazyColumn {
    item {
        // item 1
    }
    item {
        // item 2
    }
    // 添加一组 item
    list.forEach {
        item {
            // listItem
        }
    }
    // 添加一组 item 的另一种方法
    items(list) { data ->
        // listItem
    }
}
```

> [延迟列表](https://developer.android.google.cn/jetpack/compose/lists?hl=zh-cn#lazy ) 

### 常用控件

#### Text

文本框

```kotlin
// Text.kt
@Composable
fun Text(
    text: String,
    modifier: Modifier = Modifier,
    color: Color = Color.Unspecified,
    fontSize: TextUnit = TextUnit.Unspecified,
    fontStyle: FontStyle? = null,
    fontWeight: FontWeight? = null,
    fontFamily: FontFamily? = null,
    letterSpacing: TextUnit = TextUnit.Unspecified,
    textDecoration: TextDecoration? = null,
    textAlign: TextAlign? = null,
    lineHeight: TextUnit = TextUnit.Unspecified,
    overflow: TextOverflow = TextOverflow.Clip,
    softWrap: Boolean = true,
    maxLines: Int = Int.MAX_VALUE,
    inlineContent: Map<String, InlineTextContent> = mapOf(),
    onTextLayout: (TextLayoutResult) -> Unit = {},
    style: TextStyle = LocalTextStyle.current
)
```

text：文本内容

modifier：修饰符

color：文本颜色

fontSize：字体大小

fontStyle：字体风格，有正常和斜体

fontWeight：字重，范围 1 到 1000

fontFamily：字体

letterSpacing：字宽

textDecoration：文字上的装饰，例如删除线、下划线

textAlign：文字内容对齐方法

lineHeight：行高

overflow：与 softWrap 配置文本换行行为

softWrap：与 overflow 配置文本换行行为

maxLines：最大行数

inlineContent：用于在文本布局中插入可组合元素

onTextLayout：在 compose 进行布局时可通过回调提供的 TextLayoutResult 对象来对文本显示进行修改

style：文本风格，优先生效上面的配置信息

```kotlin
// 使用
// 普通文本
Text(text = "你的内容")
// 资源字符串
Text(text = stringResource(id = R.string.content))
```

> [Compose 中的文字](https://developer.android.google.cn/jetpack/compose/text?hl=zh-cn#displaying-text ) 

#### TextField

输入框

```kotlin
// TextField.kt
@Composable
fun TextField(
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    readOnly: Boolean = false,
    textStyle: TextStyle = LocalTextStyle.current,
    label: @Composable (() -> Unit)? = null,
    placeholder: @Composable (() -> Unit)? = null,
    leadingIcon: @Composable (() -> Unit)? = null,
    trailingIcon: @Composable (() -> Unit)? = null,
    isError: Boolean = false,
    visualTransformation: VisualTransformation = VisualTransformation.None,
    keyboardOptions: KeyboardOptions = KeyboardOptions.Default,
    keyboardActions: KeyboardActions = KeyboardActions(),
    singleLine: Boolean = false,
    maxLines: Int = Int.MAX_VALUE,
    interactionSource: MutableInteractionSource = remember { MutableInteractionSource() },
    shape: Shape =
        MaterialTheme.shapes.small.copy(bottomEnd = ZeroCornerSize, bottomStart = ZeroCornerSize),
    colors: TextFieldColors = TextFieldDefaults.textFieldColors()
)
```

value： 输入框中的初始文本

onValueChange：输入过程中文本变更时的回调

modifier：修饰符

enabled：是否可以获取焦点

readOnly：输入框是否为不可编辑，与 enabled 的区别是即使为不可编辑仍然可以进行选择文字复制。

textStyle：文本风格

label：显示在文本字段内的可选标签，未获得焦点时呈现

placeholder：获得焦点时的默认呈现类似Tint的效果

leadingIcon：输入框前部的图标

trailingIcon：输入框后部的图标

isError：输入内容是否错误，如果为 true，则 label，Icon 等会相应的展示错误的显示状态

visualTransformation：内容显示转变，例如输入密码时可以变成特定效果

keyboardOptions：软键盘类型

keyboardActions：等价于 android:imeAction

singleLine：是否单行

maxLines：输入框最大行数，如果 singleLine 设置为 true，这个参数将被忽略

interactionSource：与输入框的交互行为

shape：输入框的形状

colors：文本、内容(包括标签、占位符、前面和后面的图标)和背景在不同状态下的颜色，类似 Android 的 ColorStateList

```kotlin
// 使用
TextField(
    value = content, 
    onValueChange = {
    },
    shape = RoundedCornerShape(100.dp),
    colors = TextFieldDefaults.textFieldColors(
        backgroundColor = Color(LocalContext.current.getColor(R.color.gray_F8F8FA)),
        disabledIndicatorColor = Color.Transparent,
        unfocusedIndicatorColor = Color.Transparent,
        focusedIndicatorColor = Color.Transparent,
        errorIndicatorColor = Color.Transparent
    )
)
```

> [输入和修改文字](https://developer.android.google.cn/jetpack/compose/text?hl=zh-cn#enter-modify-text ) 

#### Image

图片控件

```kotlin
// Image.kt
@Composable
fun Image(
    painter: Painter,
    contentDescription: String?,
    modifier: Modifier = Modifier,
    alignment: Alignment = Alignment.Center,
    contentScale: ContentScale = ContentScale.Fit,
    alpha: Float = DefaultAlpha,
    colorFilter: ColorFilter? = null
)
```

painter：绘制的图片，通常为 [ImageBitmap 与 ImageVector](https://developer.android.google.cn/jetpack/compose/graphics/images/compare?hl=zh-cn)

contentDescription：图片内容描述，用于 Android 辅助功能

modifier：修饰符

alignment：图片对齐方式

contentScale：图片缩放方式

alpha：图片透明度

colorFilter：可通过该参数对图片进行着色

```kotlin
// 使用
// 仅支持 jpg、svg、png 等常规图片
Image(
    painter = painterResource(R.drawable.logo),
    contentDescription = null
)
// 需要支持 shape 的 drawable 场景
val drawable = AppCompatResources.getDrawable(context, R.drawable.shape)
Image(
    painter = rememberDrawablePainter(drawable = drawable),
    contentDescription = null
)
```

> [加载图片](https://developer.android.google.cn/jetpack/compose/graphics/images/loading?hl=zh-cn  )  

#### Dialog

对话框

```kotlin
// Dialog.kt
@Composable
fun Dialog(
    onDismissRequest: () -> Unit,
    properties: DialogProperties = DialogProperties(),
    content: @Composable () -> Unit
) 
```

onDismissRequest：需要退出对话框的回调，用于执行退出对话框

properties：定制对话框的交互行为

content：内容

```kotlin
// 使用
val isShow = remember {mutableStateOf(true)}
Dialog(onDismissRequest = {
    isShow.value = false
}) {
    // 你的内容
}
```

## 修饰符

修饰符可用来执行以下操作：

- 设定控件的大小、布局、行为和外观等
- 添加高级互动，如使元素可点击、可滚动、可拖动或可缩放

下面列出几个常用的方法：

```kotlin
Modifier
    .width()                  // 控件宽度
    .height()                 // 控件高度
    .fillMaxSize()            // 宽高按照一定比例填充父布局 (0~1，默认填满)
    .fillMaxWidth()           // 宽按照一定比例填充父布局 (0~1，默认填满)
    .fillMaxHeight()          // 高按照一定比例填充父布局 (0~1，默认填满)
    .padding()                // 控件边距
    .border()                 // 控件边框
    .background()             // 控件背景
    .clickable()              // 添加点击事件
    .constrainAs()            // 约束布局中设定约束
```

需要注意的是修饰符调用过程中顺序不同产生的影响不同，例如：

```kotlin
@Composable
fun MyModifier1() {
    Row(
        horizontalArrangement = Arrangement.SpaceAround,
        verticalAlignment = Alignment.CenterVertically,
        modifier = Modifier.fillMaxSize()
    ) {
        Box(
            modifier = Modifier
                .size(100.dp)
                .background(
                    color = MaterialTheme.colorScheme.primary,
                    shape = MaterialTheme.shapes.medium
                )
                .clickable {
 
                }
                .padding(20.dp)
        )
 
        Box(
            modifier = Modifier
                .size(100.dp)
                .background(
                    color = MaterialTheme.colorScheme.primary,
                    shape = MaterialTheme.shapes.medium
                )
                .padding(20.dp)
                .clickable {
 
                }
        )
    }
}
```

![](https://raw.githubusercontent.com/ALightGroup/ALightGroup.github.io/alg-img/29ae3dc8-de25-4e67-a741-f323b713c1d4.gif)

## 与其他 Jetpack 组件联动

### ViewModel

使用时需要依赖

```groovy
implementation "androidx.lifecycle:lifecycle-viewmodel-compose"
```

然后在代码中引用

```kotlin
class MyViewModel : ViewModel() { /*...*/ }
// 方式 1
@Composable
fun MyScreen(
    viewModel: MyViewModel = viewModel()
) { /* ... */ }
```

该 ViewModel 实例的生命周期跟随控件所依赖的 Activity 或者 Fragment 变化。

### LiveData

使用时需要依赖

```groovy
implementation "androidx.compose.runtime:runtime-livedata"
```

然后在代码中引用

```kotlin
val liveDataState : State<Any> = viewModel.liveData.observeAsState()
if(liveDataState.data == null){
    // show error
} else {
    // show UI
}
```

通过这种方式引用每当 LiveData 持有的数据实例变化时就会发起一次重组，从而实现界面的实时更新。

### Navigation

使用 compose 的方式进行导航跳转

```kotlin
val ROUTE1 = "Route1"
val ROUTE2 = "Route2"
val ROUTE3 = "Route3"
val navController = rememberNavController()
    NavHost(
        modifier = Modifier,
        navController = navController,
        startDestination = ROUTE1
    ) {
        composable(ROUTE1) {
            Route1UI()
        }
        composable(ROUTE2) {
            Route2UI()
        }
        composable(ROUTE3) {
            Route3UI()
        }
    }
    // 调用以下方法进行跳转
    // navController.navigate(ROUTE1)
    // navController.navigate(ROUTE2)
    // navController.navigate(ROUTE3)
}
```

需要注意的是跳转的页面大小取决于 NavHost 所依赖父布局的大小。

