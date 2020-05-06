- [Flutter开发之导航与路由管理](https://juejin.im/post/5d452cbbf265da039e129f15)
- [路由管理](https://book.flutterchina.club/chapter2/flutter_router.html)


#### Route

在Flutter开发中，实现页面跳转需要同时使用Route 和 Navigator。

- Route 是一个应用程序抽象的屏幕或页面；
- Navigator 是一个管理路由的 widget；

> 路由(Route)，在移动开发中通常用来表示移动应用的页面（Page），具体来说，Route在Android中通常指一个Activity，在iOS中指一个ViewController。
Navigator是一个路由管理的widget，它通过一个栈来管理一个路由widget集合。通常当前屏幕显示的页面就是栈顶的路由

示例: 为了说明 Flutter 是如何实现路由跳转的，我们创建两个页面：NewRoute.dart 和 main.dart。
其中，NewRoute.dart的源码如下：

```
import 'package:flutter/material.dart';
import 'package:flutter/cupertino.dart';

class SecondPage extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
          title: new Text('新页面')
      ),
      body: new Center(
        child: new Text(
          '点击浮动按钮返回首页',
        ),
      ),
      floatingActionButton: new FloatingActionButton(
        onPressed: () {
             Navigator.of(context).pop();
        },
        child: new Icon(Icons.replay),
      ),
    );
  }
}
```


main.dart的源码如下：

```
import 'package:flutter/material.dart';
import 'package:flutter_demo/SecondPage.dart';

void main() {
  runApp(new MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Flutter Demo',
      theme: new ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: new MyHomePage(title: '路由管理首页'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  
  MyHomePage({Key key, this.title}) : super(key: key);

  final String title;

  @override
  _MyHomePageState createState() => new _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text(widget.title),
      ),
      body: new Center(
        child: new Text(
          '点击浮动按钮打开新页面',
        ),
      ),
      floatingActionButton: new FloatingActionButton(
        onPressed: () {
          Navigator.push(
              context, MaterialPageRoute(builder: (context) => SecondPage()));
        },
        child: new Icon(Icons.open_in_new),
      ),
    );
  }
}
```

运行上面的代码，当我们点击按钮后，路由就会打开一个新的路由页面


#### MaterialPageRoute

MaterialPageRoute 继承自 PageRoute 类，PageRoute 类是一个抽象类，表示占有整个屏幕空间的一个模态路由页面，它还定义了路由构建及切换时过渡动画的相关接口及属性。MaterialPageRoute 是 Material 组件库的一个 Widget，它可以针对不同平台，实现与平台页面切换动画风格一致的路由切换动画，具体来说：

> 对于Android，当打开新页面时，新的页面会从屏幕底部滑动到屏幕顶部；当关闭页面时，当前页面会从屏幕顶部滑动到屏幕底部后消失，同时上一个页面会显示到屏幕上。对于iOS，当打开页面时，新的页面会从屏幕右侧边缘一致滑动到屏幕左边，直到新页面全部显示到屏幕上，而上一个页面则会从当前屏幕滑动到屏幕左侧而消失；当关闭页面时，正好相反，当前页面会从屏幕右侧滑出，同时上一个页面会从屏幕左侧滑入。

我们使用 MaterialPageRoute 来完成路由跳转时，MaterialPageRoute 构造函数提供了几个的，参数，格式如下：

```
 MaterialPageRoute({
    WidgetBuilder builder,
    RouteSettings settings,
    bool maintainState = true,
    bool fullscreenDialog = false,
  })
```

这些参数的具体含义如下：

- builder 是一个WidgetBuilder类型的回调函数，它的作用是构建路由页面的具体内容，返回值是一个widget。我们通常要实现此回调，返回新路由的实例。
- settings 包含路由的配置信息，如路由名称、是否初始路由（首页）。
- maintainState：默认情况下，当入栈一个新路由时，原来的路由仍然会被保存在内存中，如果想在路由没用的时候释放其所占用的所有资源，可以设置maintainState为false。
- fullscreenDialog表示新的路由页面是否是一个全屏的模态对话框，在iOS中，如果fullscreenDialog为true，新页面将会从屏幕底部滑入（而不是水平方向）。

#### Navigator

Navigator 是 Flutter应用开发中的一个路由管理的 widget，它通过一个栈来管理一个路由widget集合。通常，当前屏幕显示的页面就是栈顶的路由。Navigator提供了一系列方法来管理路由栈，我们可以使用 push 和 pop 两个操作来进行页面的入栈和出栈。

##### push

将给定的路由入栈（即打开新的页面），返回值是一个Future对象，用以接收新路由出栈（即关闭）时的返回数据。

执行push 操作时，我们主要使用两个方法：一个是直接 push 一个路由，另外一个是 pushNamed 一个命名路由地址。

- push方式: 下边是 Navigator.push 的源码，入参的 Route 对象中有一个 RouteSettings 成员变量，我们可以在构造 Route 对象的时候将需要传递的参数放在 RouteSettings 中。

```
@optionalTypeArgs
static Future<T> push<T extends Object>(BuildContext context, Route<T> route) {
  return Navigator.of(context).push(route);
}
```

如果涉及到传递参数，那么我们可以将参数放在 SecondScreen 的构造函数中，也可以放在构造的 MaterialPageRoute 的 RouteSettings 中。

```
Navigator.push(
  context,
  new MaterialPageRoute(builder: (context) => new SecondScreen()),
).then((data){
  //接受返回的参数
  print(data.toString());
};
```


##### pushNamed方式

pushNamed方式的实现最终调用的也是 push 方法，这中方法直接暴露了参数 Object arguments ，源码如下：

```
@optionalTypeArgs
static Future<T> pushNamed<T extends Object>(
  BuildContext context,
  String routeName, {
  Object arguments,
  }) {
  return Navigator.of(context).pushNamed<T>(routeName, arguments: arguments);
}
@optionalTypeArgs
Future<T> pushNamed<T extends Object>(
  String routeName, {
  Object arguments,
}) {
  return push<T>(_routeNamed<T>(routeName, arguments: arguments));
}
```

使用pushNamed方式时，需要将路由注册到路由表中，例如：

```
Navigator.of(context)
  .pushNamed(
    '/route1',
    arguments: {
      "name": 'hello'
    }
	).then((data){
  	//接受返回的参数
  	print(data.toString());
	};
```

##### pop

pop 操作将栈顶路由出栈，入参为一个 object 类型的对象，出参为当前页面关闭时返回给上一个页面的数据。

pop的源码如下：

```
@optionalTypeArgs
static bool pop<T extends Object>(BuildContext context, [ T result ]) {
  return Navigator.of(context).pop<T>(result);
}
```

pop 的使用非常简单，例如：

```
Navigator.of(context).pop("");  //可以传递参数
```

两个页面之间跳转，如果涉及到参数的传递，可以使用下面的方式：

```
Navigator.of(context).pushNamed('/route1', arguments: {"name": 'hello'});
```

获取参数时，可以使用下面的方式：

```
class Page extends StatelessWidget{
  String name;
  @override
  Widget build(BuildContext context) {
    dynamic obj = ModalRoute.of(context).settings.arguments;
    if (obj != null && isNotEmpty(obj["name"])) {
      name = obj["name"];
    }
    return Material(
      child: Center(
        child: Text("this page name is ${name}"),
      ),
    );
  }
}
```

#### 命名路由

所谓命名路由，就是给路由起一个名字，然后可以通过路由名字直接打开新的路由。这为路由管理带来了一种直观、简单的方式


路由名称按惯例使用类似路径的结构，应用程序的主页路由默认为“/”，例如，'/ home' 表示 HomeScreen， '/ login' 表示 LoginScreen。

##### 路由表

要想使用命名路由，我们必须先提供并注册一个路由表（routing table），这样应用程序才知道哪个名称与哪个路由Widget对应。路由表的定义如下：
`Map<String, WidgetBuilder> routes;`它是一个Map， key 为路由的名称，是个字符串；value是个builder回调函数，用于生成相应的路由Widget。我们在通过路由名称入栈新路由时，应用会根据路由名称在路由表中找到对应的 WidgetBuilder 回调函数，然后调用该回调函数生成路由 widget 并返回。


例如，我们在创建 MaterialApp 的时候就有一个 routes 构造参数：

```
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Flutter Demo',
      home: new MyHomePage(title: '应用程序首页'),
      routes: <String, WidgetBuilder> {
        '/a': (BuildContext context) => new MyPage(title: 'A 页面'),
        '/b': (BuildContext context) => new MyPage(title: 'B 页面'),
        '/c': (BuildContext context) => new MyPage(title: 'C 页面')
      },
    );
  }
}
```

##### 注册路由表

Flutter的路由注册方式比较简单，我们回到之前“计数器”的示例，然后在MyApp类的build方法中找到MaterialApp，添加routes属性，代码如下：

```
return new MaterialApp(
  title: 'Flutter Demo',
  theme: new ThemeData(
    primarySwatch: Colors.blue,
  ),
  //注册路由表
  routes:{
   "new_page":(context)=>NewRoute(),
   "/":(context) => MyHomePage(title: 'Flutter Demo Home Page'), //注册首页路由
  } ,
  initialRoute:"/", //名为"/"的路由作为应用的home(首页)
);
```

这样，使用routes的方式我们就完成了路由表的注册。现在，我们就可以通过路由名称来打开新的路由。pushNamed跳转的格式如下：

```
Future pushNamed(BuildContext context, String routeName,{Object arguments})
```

Navigator 除了pushNamed方法，还有pushReplacementNamed等其他管理命名路由的方法。接下来我们通过路由名来打开新的路由页，修改FlatButton的onPressed回调代码：

```
onPressed: () {
  Navigator.pushNamed(context, "new_page");
  //Navigator.push(context,
  //  new MaterialPageRoute(builder: (context) {
  //  return new NewRoute();
  //}));  
},
```

##### 命名路由传参

在Flutter最初的版本中，命名路由是不能进行传递参数的，后来才支持了参数。例如，下面展示命名路由如何传递并获取路由参数，首先，注册一个路由：

```
routes:{
   "new_page":(context)=>EchoRoute(),
  } ,
```

然后，在路由页通过RouteSetting对象获取路由参数，例如：

```
class EchoRoute extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    //获取路由参数  
    var args=ModalRoute.of(context).settings.arguments
    //...省略无关代码
  }
}
```

然后，在打开路由时传递参数：

```
Navigator.of(context).pushNamed("new_page", arguments: "hi");
```

#### 路由生成钩子

例：如果要实现一个电商 App，在用户没有登录时可以查看店铺，商品等信息，但交易信息，购物车需要登陆后才能查看，为此需要在打开每个路由前判断登录状态，这样非常麻烦，MaterialApp 提供了一个 onGenerateRoute 属性，它在打开命名路由时可能会被调用，为什么是可能，因为当调用 Navigator.pushNamed() 打开路由时，如果指定路由在路由表已注册，则会调用对应的 builder 函数来生成组件，如果没有注册，才会调用 onGenerateRoute 来生成路由，使用 onGestureRoute 回调，放弃路由表，可以很方便的实现统一的权限管理。

```
MaterialApp(
  ... //省略无关代码
  onGenerateRoute:(RouteSettings settings){
      return MaterialPageRoute(builder: (context){
           String routeName = settings.name;
       // 如果访问的路由页需要登录，但当前未登录，则直接返回登录页路由，
       // 引导用户登录；其它情况则正常打开路由。
     }
   );
  }
);
```

> onGenerateRoute 只会对命名路由生效
