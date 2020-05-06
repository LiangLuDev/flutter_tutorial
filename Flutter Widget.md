- [Flutter学习指南：UI布局和控件](https://juejin.im/post/5bd54b7be51d456c430e35f6)
- [Flutter快速上车之Widget](https://juejin.im/post/5b8ce76f51882542c0626887)
- [Flutter 布局常用的 widgets（Common layout widgets）](https://www.jianshu.com/p/fccb4c43c268)
- [Flutter widget doc](https://flutter.dev/docs/reference/widgets)
- [Flutter 实战-基础组件](https://book.flutterchina.club/chapter3/flutter_widget_intro.html)

## 基础概念

Flutter: 一切皆组件，组件大致有如下分类

- 基础组件：常用的有文本，按钮，图片，单选框，复选框，输入框，进度指示器
- 布局类组件：线性布局（Row Column）弹性布局（Flex）流式布局（Wrap）层叠布局（Stack Positioned）对齐（Align）
- 容器类组件：填充（Padding）尺寸限制容器（ConstrainedBox）装饰容器（DecoratedBox）变换（Transform）Container 容器，Scaffold TabBar BottomNavigationBar，裁剪（Clip）
- 可滚动组件：SingleChildScrollView，ListView，GridView，CustomScrollView，滚动监听控制（ScrollController）
- 功能型组件：导航返回拦截（WillPopScope）数据共享（inheritedWidget）跨组件状态共享（Provider）颜色和主题（Theme）异步UI（FutureBuilder，StreamBuilder），对话框


## Widget 的分类

Widget 分为 StatelessWidget 和 StatefulWidget，它们都继承自 Widget，Widget 类的声明如下：

```
@immutable
abstract class Widget extends DiagnosticableTree {
  const Widget({ this.key });
  final Key key;

  @protected
  Element createElement();

  @override
  String toStringShort() {
    return key == null ? '$runtimeType' : '$runtimeType-$key';
  }

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
}
```

- Widget 类继承自 DiagnosticableTree, 即“诊断树”，用来提供 DEBUG 信息
- Key：这个 Key 的作用主要是再下一次 build 时复用旧的 Widget， 决定条件在 canUpdate() 方法中。
- createElement(): 如上所说，一个 Widget 可以对应多个 Element， Flutter Framework 在构建 UI 树时，会先调用此方法生成对应节点的 Element 对象，这个方法由 Framework 调用，我们一般不会用到。
- debugFillProperties(): 复写父类的方法，主要设置诊断树的一些特征。
- canUpdate(): 静态方法，主要用于在 Widget 树重新 build 时复用旧的 Widget： 是否用新的 Widget 对象去更新旧的 UI 树上所对应的 Element 对象的配置，只要 newWidget 与 oldWidget 的 runtimeType 和 key 同时相等，就会用 newWidget  的配置去更新 Element 对象的配置，否则就会创建新的 Element

实际使用时，我们一般不直接继承 Widget 类，而是继承它的子类 StatelessWidget 和 StatefulWidget

> 一些概念：Flutter 中真正在屏幕上显示的类是 Element， Widget 只是描述 Element 的配置数据，一个 Widget 可以对应多个 Element，因为同一个 Widget 对象可以被添加到 UI 树的不同部分，渲染时，每个 Element 节点都会对应一个 Widget 对象。Widget 和 Element 的关系类似类和对象。


#### StatelessWidget

StatelessWidget 继承自 Widget

```
abstract class StatelessWidget extends Widget {
  const StatelessWidget({ Key key }) : super(key: key);

  @override
  StatelessElement createElement() => StatelessElement(this);

  @protected
  Widget build(BuildContext context);
}
```

StatelessWidget 用于不需要维护状态的场景，它通常在 build 方法中通过嵌套其它 Widget 来构建 UI。一个小例子：显示一个带灰色背景的文本

```
class Echo extends StatelessWidget {
  const Echo({
    Key key,  
    @required this.text,
    this.backgroundColor:Colors.grey,
  }):super(key:key);

  final String text;
  final Color backgroundColor;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        color: backgroundColor,
        child: Text(text),
      ),
    );
  }
}
```

几个需要注意的点：

- Widget 的构造函数参数应当使用命名参数，命名参数中的必要参数需要添加 @required 标注，有利于静态代码分析
- Widget 在继承时，第一个参数通常应当是 Key
- 如果 Widget 需要接收子 Widget， 那么 child 或 children 应当被放在参数列表的最后。
- Widget 的属性应当被声明为 final，防止被意外修改

在其他组件中这样来使用上面的 widget

```
Widget build(BuildContext context) {
    return Echo(text: "hello")
}
```

#### StatefulWidget

StatefulWidget 也继承自 Widget 类并重写了 createElement() 方法，它和 StatelessWidget 返回的 Element 对象不同，另外它还新增了一个接口 createState()

```
abstract class StatefulWidget extends Widget {
  const StatefulWidget({ Key key }) : super(key: key);

  @override
  StatefulElement createElement() => new StatefulElement(this);

  @protected
  State createState();
}
```

createState() 方法用来创建 StatefulWidget 的 State，State 表示与其对应的 StatefulWidget 要维护的状态，State 中的状态在 Widget 构建时可以读取，如果其中的状态发生了改变，我们可以调用 setState 方法通知 framework 来重新调用其 build 方法重新构建 widget 树，达到刷新 UI 的目的。

State 对象中有两个常用属性：

- widget：表示与该 State 实例关联的 widget 实例。注意这种关联并非永久，在应用生命周期中， UI 树上的某一个 Widget 实例在重新构建时可能会发生变化，但 State 实例只会在第一次插入到树中时创建，当 widget 被修改了，State.widget 会被设置为新的 widget 实例。
- context：上下文对象，可用于树的遍历和其他一些操作。


State 的生命周期

![image](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget1.png?raw=true)

- initState：当Widget第一次插入到Widget树时会被调用，对于每一个State对象，Flutter framework只会调用一次该回调，所以，通常在该回调中做一些一次性的操作，如状态初始化、订阅子树的事件通知等。
- didChangeDependencies()：当State对象的依赖发生变化时会被调用。典型场景是当系统语言 Locale 或者应用主题改变时，framework 会回调此方法。
- build(): 用于构建 widget 子树。
- reassemble(): 专为开发调试提供，在热重载时会被调用。
- didUpdateWidget(): 在 widget 重新构建时，Flutter framework会调用 Widget.canUpdate 来检测 Widget 树中同一位置的新旧节点，然后决定是否需要更新，如果 Widget.canUpdate 返回 true 则会调用此回调。
- deactivate(): 当 State 对象从树中被移除时，会调用此回调。在一些场景下，Flutter framework会将State对象重新插到树中，如包含此 State 对象的子树在树的一个位置移动到另一个位置时（可以通过 GlobalKey 来实现）。如果移除后没有重新插入到树中则紧接着会调用 dispose() 方法。
- dispose(): 当 State 对象在树中被永久移除时调用，通常在此释放资源。

**State 对象里包含了大量逻辑操作，我们可能需要调用其中暴露出来的某些方法，如何获取其他 Widget 的 State 对象？**

**方法一**：通过 Context 获取，context 对象有一个 ancestorStateOfType(TypeMatcher) 方法，该方法可以从当前节点沿着 widget 树向上查找指定类型的 StatefulWidget 对应的 State 对象。

例：获取 Scaffold 的 State 对象，然后显示 SnackBar，这里有个坑，直接在 Scaffold 方法里面调用是获取不到的，如需要把 RaiseButton 提取出来

```
class HomePage extends StatefulWidget {
  HomePage(Key key): super(key: key);

  @override
  State<StatefulWidget> createState() => new _HomePageState();
}

class _HomePageState extends State<HomePage> {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(title: new Text('Home'),),
      body: SingleChildScrollView(
        child: Column(
          children: <Widget>[
            MyButton()
          ],
        ),
      ),
    );
  }
}

class MyButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return RaisedButton(
      onPressed:() {
//                ScaffoldState _state = context.ancestorStateOfType(
//                    TypeMatcher<ScaffoldState>());
        // 根据约定，如果 StatefulWidget 的状态是希望暴露出来的，应当在 StatefulWidget 中提供一个 of 静态方法来获取其 State
        ScaffoldState _state = Scaffold.of(context);
        _state.showSnackBar(
            SnackBar(
              content: Text("SnackBar"),
            )
        );
      },
      child: Text("Show SnackBar"),
    );
  }
}
```

**方法二**：通过 GlobalKey 获取，有两个步骤

- 1.给目标 StatefulWidget 添加 GlobalKey

```
//定义一个globalKey, 由于GlobalKey要保持全局唯一性，我们使用静态变量存储
static GlobalKey<ScaffoldState> _globalKey= GlobalKey();
...
Scaffold(
    key: _globalKey , //设置key
    ...  
)
```

- 2.通过 GlobalKey 来获取 State 对象的方法

```
_globalKey.currentState.xxxx()
```


## 组件的使用

创建 `basic_widget` 工程,接下来创建 `page_basic.dart` 文件来熟悉常见的基本组件，代码框架的内容如下

```
class PageBasic extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => new _PageBasicState();
}

class _PageBasicState extends State<PageBasic> {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(title: new Text('Basic widget')),
      body: SingleChildScrollView(  // 使用这个组件包裹我们的内容，超过一屏幕后可以滑动
        child: Container(
          child: new Column(
            children: <Widget>[
                    // 这里写我们的基本组件
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

- 文本：`Text`

```
Text(
    'Text Widget',
    maxLines: 1,  // 最多多少行
    overflow: TextOverflow.ellipsis, // 溢出显示 ...
    style: TextStyle(   // 添加样式
        decoration:TextDecoration.underline,  // 添加下划线，还可以设置中划线，上划线，一般使用 none
        fontSize: 30.0, // 文字大小
        color: Colors.blue,
        fontWeight: FontWeight.bold),
),

Text.rich(TextSpan( // 富文本
  children:[
    TextSpan(
      text: "Site:"
    ),
    TextSpan(
      text:"flutter.com",
      style: TextStyle(
        color: Colors.blue
      ),
      recognizer: _tapRecognizer..onTap = (){
        print("Tap link");
      },
    )
  ]
)),

DefaultTextStyle(   //  指定文本默认样式
  style: TextStyle(
    color: Colors.red,
    fontSize: 20.0
  ),
  textAlign: TextAlign.center,
  child: Column(
    children: <Widget>[
      Text("Hello Flutter"),
      Text("I am Ramon"),
      Text("I like programing",
        style: TextStyle(
          inherit: false, // 不继承默认样式
          color: Colors.blue
        ),
      )
    ],
  ),
),
```

- 按钮： `FlatButton`  `RaiseButton` `OutlineButton`, 它们的使用方法类似，都可以设置 `onPressed` 点击事件，通过 `child` 参数设置内容（可以是任意 widget），比如可以是文本，也可以是图片。它们的区别只有显示效果不同。

```
RaisedButton.icon(  // 带 icon 的 button
  icon:Icon(Icons.send),
  label: Text("RaisedButton"),
  onPressed: (){},
),
FlatButton(   // 自定义 Button 外观
  color: Colors.blue,
  highlightColor: Colors.blue[700],
  colorBrightness: Brightness.dark, // 指定按钮主题，设置文字颜色为浅色
  splashColor: Colors.grey,
  child: Text("FlatButton"),
  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20)),
  onPressed: (){},
),
OutlineButton(
  child: Text("OutlineButton"),
  onPressed: (){},
),
IconButton(
  icon: Icon(Icons.thumb_up),
  onPressed: (){},
),
```

> Tip: onPress 不设置或者设置为 null 时，按钮为 disable 状态


- 图片：`Image`, 有 4 种加载图片的方法。

```
Image.asset(name);   // 会打包在包里
Image.file(file);    // 加载本地图片，不会打包在包里
Image.memory(bytes); // 加载 Uint8List, 比如可以加载 Android 应用的图标
Image.network(src);
```

加载网络图片

```
Image.network(
    "https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=4135477902,3355939884&fm=26&gp=0.jpg",
    width: 400.0,
    height: 300.0,
    fit: BoxFit.fill,
)
```

加载 `asset` 目录下的图片，首先需要在 `pubspec.yaml` 文件中声明文件的位置

```
assets:
    - images/c0.jpg
```

接下来我们才可以使用它

```
Image.asset(
  'images/c0.jpg',
  width: 200,
  height: 100,
  fit: BoxFit.cover)
```

大家可能注意到上面都使用了一个 `fit` 参数，它的作用类似与 Android 中的 `scaleType`, [BoxFit 官方文档](https://api.flutter.dev/flutter/painting/BoxFit-class.html)

- contain: 包裹图片并且尽可能的大。

![flutter003-contain](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget2.png?raw=true)

- cover: 不变形的铺满整个容器，这个用的比较多

![flutter003-cover](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget3.png?raw=true)

- fill：填充满整个容器，图片会变形。

![flutter003-fill](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget4.png?raw=true)

- fitHeight:  高度铺满，水平方向上不管是不是超过容器边界。

![flutter003-fillheight](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget5.png?raw=true)


- fillWidth: 宽度铺满，竖直方向上不管是不是超过容器边界。

![flutter003-fillWidth](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget6.png?raw=true)

- none: 默认放在容器中间，超出容器范围不显示。

![flutter003-none](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget7.png?raw=true)

- scaleDown: 默认放在容器中间，超出容器的话会压缩全部显示在容器里。

![flutter003-scaleDown](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget8.png?raw=true)


如何显示一个圆形图片，下面介绍两种方式

- 第一种：通过 Container 实现

```
//  实现圆形图片
  Container(
    width: 300,
    height: 300,
    decoration: BoxDecoration(
      borderRadius: BorderRadius.circular(150),
      image: DecorationImage(
          image: NetworkImage(
            'https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=4135477902,3355939884&fm=26&gp=0.jpg',
          ),
          fit:BoxFit.cover
      )
    ),
  ),
```

- 第二种：通过 `ClipOval` 组件

```
ClipOval(
    child: Image.network(
      'https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=4135477902,3355939884&fm=26&gp=0.jpg',
      width: 100,
      height: 100,
      fit: BoxFit.cover,
    ),
  ),
```

实际项目从网络加载图片，如果网速比较慢，我们需要显示一个 `placeholder`, `FadeInImage` 可以在加载时显示 `placeHolder`

```
FadeInImage.assetNetwork(
  placeholder: 'loading',
  image:
      'https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=4135477902,3355939884&fm=26&gp=0.jpg'),
```


- 文本输入框：`TextField` 需要通过给 `TextField` 设置一个 `controller` 来获取文本,点击按钮我们就能在控制台看到文本框中的输出。

```
class _MessageFormState extends State<MessageForm> {
  // 设置一个 controller，通过这个 controller 拿到文本框里的内容
  var editController = TextEditingController();
  @override
  Widget build(BuildContext context) {
     ...
     // 文本输入框
      TextField(
        controller: editController,
      ),
      RaisedButton(
        child: Text("click"),
        onPressed: () =>
            print("text inputted is ${editController.text}"),
      ),
      ...
  }
  
  @overrride
  void dispose() {
      super.dispose();
      // 手动调用 controller 的 dispose 方法以释放资源
      editController.dispose();
  }
}
```

接下来我们使用一个弹框来显示上面文本域输入的内容，注意弹框是和用户有交互的，所以需要显示在 `StateFulWidget` 中，改造上面的代码

```
RaisedButton(
      child: Text("click"),
      onPressed: () {
        print("text inputted is ${editController.text}");
        showDialog(
          // 第一个 context 是参数名字，第二个 context 是 State 的成员变量
            context: context,
            builder: (_) {
              return AlertDialog(
                // dialog 的内容
                content: Text(editController.text),
                // action 设置 dialog 的按钮
                actions: <Widget>[
                  FlatButton(
                    child: Text("Ok"),
                    // 用户点击按钮后，关闭弹框
                    onPressed: () => Navigator.pop(context),
                  )
                ],
              );
            }
        );
      }
  ),
```


- Container: 可以通过它设置控件的尺寸，背景，margin 等。
    - Decoration: 一般使用它的子类 BoxDecoration, 使用它可以设置 Container 的背景色/边框/圆角/阴影/渐变等功能。
    - BoxConstraints: 用来描述组件的大小。Flutter 中 `double.infinity` 用来表示尽可能大的意思。
    - transform: 可以进行位移/变形/旋转
    - alignment: 设置内容位置。

```
Container(
  child: Text('Container'),
  padding: EdgeInsets.all(8.0),
  margin: EdgeInsets.all(4.0),
  width: 80.0,
  decoration: BoxDecoration(
    color: Colors.yellow,
    borderRadius: BorderRadius.all(Radius.circular(10))
  ),
),
```

- Padding: 如果只需要 padding，可以单独使用 Padding 组件。

```
Padding(
  child: Text('Padding'),
  padding: EdgeInsets.all(20),
),
```

- Center: 将一个控件放到中间

```
Container(
    padding: EdgeInsets.all(8.0),
    margin: EdgeInsets.all(4.0),
    width: 200.0,
    decoration: BoxDecoration(
        color: Colors.red,
        borderRadius: BorderRadius.all(Radius.circular(10))),
    // 把文本放到 Container 中间
    child: Center(
      child: Text('Center widget'),
    ),
  ),
```


- Row: 水平布局，Column：竖直布局。（也就是 Flex 布局加上 direction 参数，它们都继承自 Flex 布局），其中有几个常用属性
    - 主轴和纵轴：如果布局是水平排列的，那么主轴就是水平方向，而纵轴是竖直方向，布局垂直排列与之类似。` MainAxisAlignment` 和 `CrossAxisAlignment` 分别代表主轴对齐和纵轴对齐。
    - `textDirection`: 表示水平方向子组件的布局顺序（从左到右或从右到左），默认是系统当前文字显示的方向。
    - `maxAxisSize`:表示在水平方向占据的宽度，默认是 `MainAxisSize.max`,表示占据京可能多的空间。如果设置为 `MainAxisSize.min` 则表示占用尽可能少的空间，等于所有子组件的空间之和。
    - `mainAxisAlignment`: 表示在主轴的对齐方式，如果 `mainAxisSize` 的值为 `min`,这个属性没有意义。 `start` 表示沿着 `textDirecton` 初始方向对齐，`end` 和 `start` 相反，`center`表示居中。
    - `verticalDirection`: 竖直方向，和上面类似。
    - `crossAxisAlignment`: 竖直方向，和上面类似。
    - `Expand`: 占满一行或者一列的剩余空间，如果有多个 `Expand`，可以通过 `flex` 参数设置比例。

```
Row({
  ...  
  TextDirection textDirection,    
  MainAxisSize mainAxisSize = MainAxisSize.max,    
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
  VerticalDirection verticalDirection = VerticalDirection.down,  
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
  List<Widget> children = const <Widget>[],
})
```



```
Flex(
    direction: Axis.horizontal,     // 相当于 Row， Row 和 Column 都是继承自 Flex
    mainAxisAlignment: MainAxisAlignment.spaceEvenly,
    crossAxisAlignment: CrossAxisAlignment.center,
    children: <Widget>[
      new Flexible(
        flex: 2,
        fit: FlexFit.loose,
        child: new Container(
          color: Colors.blue,
          height: 60.0,
          alignment: Alignment.center,
          child: const Text(
            'left',
            textAlign: TextAlign.center,
            style: TextStyle(color: Colors.black),
            textDirection: TextDirection.ltr,
          ),
        ),
      ),
      new Flexible(
          flex: 1,
          fit: FlexFit.loose,
          child: new Container(
            color: Colors.red,
            height: 60.0,
            alignment: Alignment.center,
            child: const Text(
              'right',
              textAlign: TextAlign.center,
              style: TextStyle(color: Colors.black),
              textDirection: TextDirection.ltr,
            ),
          )),
    ],
  ),
  Row(
    children: <Widget>[
      Expanded(
        // 占一行的 2/3
        flex: 2,
        child: RaisedButton(
          child: Text('btn1'),
        ),
      ),
      Expanded(
        // 占一行的 1/3
        flex: 1,
        child: RaisedButton(
          child: Text('btn2'),
        ),
      ),
    ],
  ),
```

- `Stack`: 类似 `FrameLayout`, 默认情况下是左上角对其的，我们可以通过设置 `alignment` 参数来改变这个对齐的位置。`Stack` 的子 `widget` 分为两类，`Positiond` 组件和位置 `Positioned` 的组件。
    - Positioned 组件，可以通过 Positioned 的属性灵活定位，类似与 absolute 布局，通过设置 left/top/right/bottom 的距离来设置位置。
    - 未指定 Positioned 的组件可以通过 `Stack` 的 alignment 属性指定其位置。
    - 另外我们还可以通过 `Align` 组件来指定组件的位置。

```
Container(
width: double.infinity,
height: 300,
child: Stack(
    // Alignment 取值范围为 [-1, 1], Stack 中心为 (0, 0)
    // 下面设置 (-0.5, -0.5) 后，可以让文本对齐到 Container 的 1/4 处
    alignment: const Alignment(0.5, 0.5), // 指定未定位组件的位置
    fit:StackFit.expand,  // 未定位组件占满 Stack 整个空间
    children: <Widget>[
      Container(
        child: Text("Hello world",style: TextStyle(color: Colors.white)),
        color: Colors.red,
      ),
      Positioned(
        left: 0,
        top: 0,
        child: Text('Stack text 1'),
      ),
      Positioned(
        right: 0,
        top: 0,
        child: Text('Stack text 2'),
      ),
      Align(
        alignment: Alignment.center,
        child: Icon(Icons.home, color: Colors.white,size: 40),
      ),
    ],
  ),
)
```

- `GestureDetector`: Flutter 中手势操作也是一个 widget，使用时只需要将它包裹在目标 widget 的外面，然后实现相应的事件，包括点击/长按/缩放/拖动等。

```
GestureDetector(
    child: Container(
      alignment:Alignment.center,
      width: 200,
      height:200,
      color:Colors.blue,
      child: Text(
          _operation,
        style: TextStyle(
          color: Colors.white
        )
      ),
    ),
    onTap: ()=>updateText('Tap'), // 点击
    onDoubleTap: ()=>updateText('DoubleTab'),
    onLongPress: ()=>updateText('LongPress'),
  )
```

创建 `updateText` 方法并且定义一个 `_operation` 变量

```
String _operation = "No Gesture detected";

void updateText(String text) {
// 更新事件名字
setState((){
  _operation = text;
});
}
```

- `AspecRatio`: 可以指定子元素的宽高比，并让其尽可能的铺满父元素,也就是约束子元素的宽高比例, 下面例子的图片就是以 `16 / 9` 显示。

```
Container(
    width: 400,
    child:AspectRatio(
        aspectRatio: 16.0 / 9.0, // 设置宽高比为 16 ：9
        child: Image.asset('images/c0.jpg', fit: BoxFit.cover,),
    ),
  )
```

> 设置不合理的宽高，比如我们设置宽高都为 100，然后设置 ratio 为 0.5, 则计算时会以高为标准，算出来宽是 50

- `Card` 组件：卡片布局，可以实现阴影效果。

![flutter003-Card](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget9.png?raw=true)

```
SizedBox(
    height: 210,
    child: Card(
      elevation: 15.0,  // 设置阴影
      shape: const RoundedRectangleBorder(borderRadius: BorderRadius.all(Radius.circular(14.0))),
      child: new Column(
        children: <Widget>[
          new ListTile(
            title: Text('标题', style: TextStyle(
              fontWeight: FontWeight.w600
            ),),
            subtitle: Text('子标题'),
            leading: Icon(
              Icons.restaurant,
              color: Colors.blue[500],
            ),
          ),
          Divider(),
          ListTile(
            title: Text('内容一', style: TextStyle(fontWeight: FontWeight.w500),),
            leading: Icon(Icons.contact_phone, color:Colors.blue[500],),
          ),
          ListTile(
            title: Text('内容二', style: TextStyle(fontWeight: FontWeight.w500),),
            leading: Icon(Icons.contact_mail, color:Colors.blue[500],),
          )
        ],
      ),
    ),
  )
```



- `Wrap` 组件，一行放不下时可以换到下一行。

![flutter003-wrap](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget10.png?raw=true)


```
SizedBox(
    height: 300.0,
    child: Wrap(
      children: <Widget>[
        createBook('斗罗大陆'),
        createBook('斗破苍穹'),
        createBook('盘龙'),
        createBook('遮天'),
        createBook('神墓'),
        createBook('诛仙'),
        createBook('凡人修仙传')
      ],
    ),
  )
  
Widget createBook(String name) {
    return Container(
      margin: EdgeInsets.only(left: 10,top: 10, right: 10),
      child: Text(name ,style: TextStyle(color: Colors.blue, fontWeight: FontWeight.w600),),
      padding: EdgeInsets.all(7),
      decoration: BoxDecoration(
        border:Border.all(
          color: Colors.blue,
          width: 2.0
        ),
        color: Colors.white,
        borderRadius: BorderRadius.all(Radius.circular(10))
      ),
);
```

- Scaffold：可以帮我们快速实现一个包含导航栏，抽屉菜单以及底部 Tab 导航的页面,它可以包含以下几个组件
    - AppBar：一个导航栏骨架，通过 TabBar 可以生成 tab 菜单
    - Drawer：抽屉菜单
    - BottomNavigationBar：底部导航栏
    - FloatingActionButton：悬浮按钮

- `BottomNavigationBar`: 底部导航栏, 接下来我们把刚才布局 body 中的内容抽离出去，新建 `basic_widget.dart` 组件，在我们的文件中使用 `BottomNavigationBar`组件

```
class PageBasic extends StatefulWidget {
  PageBasic(Key key): super(key: key);

  @override
  State<StatefulWidget> createState() => new _PageBasic();
}

class _PageBasic extends State<PageBasic> {
  int _currentIndex = 0;    // 记录当前点击的位置

  List _pageList = [
    new BasicWidget(Key('basic_widget')),       // 我们刚才抽离出去的内容
    new ScrollWidget(Key('scroll_widget'))
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('基本组件'),
      ),
      body: _pageList[_currentIndex],       // index 更新时，页面组件也跟着更新
      bottomNavigationBar: BottomNavigationBar(
          onTap: (int index){
            setState(() {
              _currentIndex = index;
            });
          },
          currentIndex: _currentIndex,
          items: [
            BottomNavigationBarItem(
              icon: Icon(Icons.home),
              title: Text('基本组件')
            ),
            BottomNavigationBarItem(
                icon: Icon(Icons.home),
                title: Text('列表组件')
            )
      ]),
    );
  }
}
```

> 当底部有多个按钮时，会发现部分消失了，我们需要配置 `BottomNavigationBar` 的 `type` 属性为 `BottomNavigationBarType.fixed`

![image](https://github.com/LiangLuDev/flutter_tutorial/blob/master/images/flutter_widget11.png?raw=true)

看下上面的效果，我們創建了一個 NavigationBar，包括兩列，基本组件和列表组件，下面我们来添加 ScrollWidget 的内容


```
Scrollable ({
    this.axisDireciton = AxisDirection.down,
    this.controller,
    this.physics,
    @required this.viewportBuilder
})
```

- Scollable：可滚动组件都直接或者间接包含一个 Scollable 组件，因此它们有一些共同属性
    - axisDirection: 滚动方向
    - physics：接收一个 ScrollPhysics 对象，它决定组件如何相应用户操作，如华东到边界时如何显示，默认情况下， Flutter 会根据具体平台分别使用不同的 ScrollPhysics 对象
        - ClampingScrollPhysics：Android 微光效果
        - BouncingScrollPhysics：IOS 弹性效果
    - controller：接收一个 ScrollController 对象，主要作用时控制滚动位置和监听滚动事件。默认有一个 PrimaryScrollController，如果子树的可滚动组件没有显式的指定 controller，并且 primary 属性值为 true，可滚动组件会使用这个默认的 PrimaryScrollController


- ScrollBar：将此组件作为滚动组件的父组件就可以添加滚动条。

- SingleChildScrollView: 只能接收一个子组件，通常只应该在展示的内容不超过屏幕太多时使用，因为 SingleChildScrollView 不支持基于 Sliver 的延迟实例化模型，所以超出屏幕尺寸太多的话它的性能比较差。


```
ListView({
  ...  
  //可滚动widget公共参数
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController controller,
  bool primary,
  ScrollPhysics physics,
  EdgeInsetsGeometry padding,

  //ListView各个构造函数的共同参数  
  double itemExtent,
  bool shrinkWrap = false,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  double cacheExtent,

  //子widget列表
  List<Widget> children = const <Widget>[],
})
```

- ListView:  它的构造函数的上半部分属于可滚动组件的公共参数，下半部分属于 ListView 构造函数共同参数
    - itemExtent：指定 item 滚动方向上的宽度，在 ListView 中，指定 itemExtent 比让子组件自己决定自身长度会更高效。
    - shrinkWrap：表示是否根据子组件的长度来设置 ListView 的长度，默认值 false
    - addAutomaticKeepAlives: 如果将列表项包裹在 AutomaticKeepAlive 中，列表项划出视口也不会被 GC，它会使用 KeepALiveNotification 保存状态。如果列表项自己维护状态，这个属性需要为 false
    - addRepaintBoundaries: 当可滚动组件滚动时，将列表项包裹在 RepaintBoundary 中可以避免列表项重绘。如果重绘代价小，例如只是一个文本，则不用加。

默认构造函数：有一个 children 参数，适合只有少量的子组件的情况。

```
ListView(
  shrinkWrap: true, 
  padding: const EdgeInsets.all(20.0),
  children: <Widget>[
    const Text('I\'m dedicating every day to you'),
    const Text('Domestic life was never quite my style'),
    const Text('When you smile, you knock me out, I fall apart'),
    const Text('And I thought I was so smart'),
  ],
);
```

ListView.builder:  适合较多数据，只有当子组件真正显示的时候才会被创建。

```
ListView.builder({
  ...
  @required IndexedWidgetBuilder itemBuilder,
  int itemCount,
  ...
})
```

ListView.separated: 可以在生成的列表项之间添加一个分割组件，比 Listview.builder 多了一个 separatorBuilder  参数。

```
// 添加红蓝相间分割线
Widget divider1 = Divider(color: Colors.blue);
Widget divider2 = Divider(color: Colors.red);
ListView.separated(
    shrinkWrap: true,
    itemCount: 5,
    // 列表项构造器
    itemBuilder: (BuildContext context, int index) {
      return ListTile(title: Text("$index"));
    },
    // 分割器构造器
    separatorBuilder: (BuildContext context, int index) {
      return index%2 == 0 ? divider1:divider2;
    },
)
```

- ListView 中一般使用 ListTile 作为列表项，可以设置 leading 前图标， Trailing 后图标，title/subtitle 以及其他许多属性。

- GridView 的构造函数，它的参数和 ListView 大部分相同，我们需要关注 gridDelette  参数，类型是  SliverGridDeletegate，它的作用是控制 GridView 子组件如何排列，SliverGridDelegate 是一个抽象类，flutter 提供了两个它的实现类。SliverGridDelegateWithFixedCrossAxisCount和SliverGridDelegateWithMaxCrossAxisExtent

```
// 实现了一个横轴为固定数量子元素的算法
SliverGridDelegateWithFixedCrossAxisCount({
  @required double crossAxisCount,  // 横轴子元素的数量
  double mainAxisSpacing = 0.0,     // 主轴方向的间距
  double crossAxisSpacing = 0.0,    // 横轴方向子元素的间距
  double childAspectRatio = 1.0,    // 子元素横轴长度和主轴长度的比例
})
```

> SliverGridDelegateWithFixedCrossAxisCount 的简写方式为 GridView.count

```
SliverGridDelegateWithMaxCrossAxisExtent({
  double maxCrossAxisExtent,    // 子元素在横轴上的最大长度
  double mainAxisSpacing = 0.0,
  double crossAxisSpacing = 0.0,
  double childAspectRatio = 1.0,
})
```

> SliverGridDelegateWithMaxCrossAxisExtent  的简写方式为 GridView.extent

上面两种方式都只适用于少量元素，大量元素推荐使用 Builder

```
 GridView.builder(
 gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
   crossAxisCount: 2,
   childAspectRatio: 1
 ),
 itemCount: icons.length,
 itemBuilder: (context, index){
   return icons[index];
 }),
```

- CustomScrollView: 需求：一个页面顶部是 Gridview，接着是 ListView，如何让他们一起滚动，CustomScrollView 可以将 SliverList SliverGird 拼接起来实现统一的滑动效果。它们和 ListView GridView 的区别是不包含滚动模型，自身不能滚动，所以可以用 CustomScrollView 把它们“粘”在一起共用 CustomScrollView 的 Scrollable。


- ScrollController: 滚动监听及控制，ScrollController 常用属性有 offset（当前滚动位置），常用方法： jumpTo（double offset） animateTo（double offset，...）  跳转到指定位置， ScrollControll 可以监听滚动

```
controller.addListener(()=>print(controller.offset))
```


- WillPopScope: 可拦截用户点击返回键，点击返回键时它的 onWillPop  方法会被调用。

```
const WillPopScope({
  ...
  @required WillPopCallback onWillPop,  // 返回一个Future 对象， false 表示不出栈， true 表示出栈
  @required Widget child
})
```

- InheritedWidget：可以提供了一种数据在 Widget 树中从上到下传递，共享的方式。（Provider 是基于此实现的）

#### Widget 可见性

- 方法一：将组件从 renderTree 中移除。
    - 单个组件隐藏自己，在 build 方法中返回一个空的 Container。
    ```
    @override
    Widget build(BuildContext context) {
    return isVisible
      ? Widget //真的Widget
      : new Container(); //空Widget 仅仅占位 并不显示
    }
    ```
    - 多个组件，在父容器的 children 的 list 中删除对应的 cell。

- Offstage: 这个组件如果它的 offstage 属性为 true，那么 Offstage 以及它的 child 组件都不会绘制到界面上。
```
@override
Widget build(BuildContext context) {
  return new Offstage(
          offstage: !isVisible,
          child:child);
}
```

- 设置透明度：通过 opacity 属性，但是不建议使用，因为设置透明度本身需要计算浪费资源而且组件占据的位置还在。
