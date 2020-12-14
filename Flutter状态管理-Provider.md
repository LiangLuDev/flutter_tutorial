[Provider项目地址](https://github.com/rrousselGit/provider)
### 状态管理 - Provider
Provider是Flutter的语法糖。不同页面使用同一个变量状态，当这个状态发生变化，依赖此状态的UI随之改变

### 如何使用

#### 创建ChangeNotifier
属于`Foundation`包下的一个类，权限比较高。主要是用来订阅它的状态变化。在值有变化时，调用`notifyListeners`，通知所有订阅此`ChangeNotifier`的对象，更新状态。

```
class UserModel with ChangeNotifier{

  String get userName => _userName;
  String _userName = 'USER_NAME';

  set userName(String value) {
    _userName = value;
    notifyListeners();
  }
}
```

#### 订阅ChangeNotifier(初始化)
```
// 单个订阅
ChangeNotifierProvider(
        create: (context) => UserModel(),
        child:Widget());
        
// 多个订阅
MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => UserModel1()),
        ChangeNotifierProvider(create: (context) => UserModel2()),
      ],
      child: Widget(),);
```
#### 使用
根据场景选择不同的方式，以下三种方式均可实现功能。

##### Provider.of
UserModel更新时，会rebuild整个Widget树。使用场景，整个Widget树使用model较多时。

```
@override
Widget build(BuildContext context) {
    UserModel model = Provider.of<UserModel>(context);
}
```
##### Consumer(局部刷新)
Consumer是一个StatelessWidget。UserModel更新时，只有某些子控件需要更新，即使用Consumer，只刷新相关的子控件，不需要全部刷新。

```
Consumer<UserModel>(
    builder: (context,model,child){
            return Text('${model.userName}');
        })

```
##### Selector(过滤刷新)
3.1版本新功能。一个 Provider 可能会为多个控件提供不同的数据，但是调用`notifyListeners`，相关控件就会刷新，而有些值与控件无关，不需要刷新。

```
Selector<GoodsModel, bool>(
  selector: (context, goodsModel) => goodsModel.isCollect,
  shouldRebuild: (pre, next) => pre != next, // 可以省略
  builder: (context, isCollect, child) => Text(isCollect)
);
```
### 为了性能

#### 尽量使用 StatelessWidget 替代 StatefulWidget
状态相关交给Provider管理。StatelessWidget 的维护成本比 StatefulWidget 要低，构建效率更高。同时更少的代码量会让我们更容易地控制重建范围，提高渲染效率。

#### 尽量使用 Consumer 替代 Provider.of(context)
Provider.of 刷新的成本是非常高，使用Consumer局部刷新对于渲染性能更好。

#### 尽量使用 Selector 替代 Consumer
Selector这种针对性刷新，比Consumer更节省性能。

#### Provider.of(context) 的隐藏属性 listen
有些场景不需要监听并刷新数据，只需要调用方法即可。这样状态更新时，不需要刷新widget。

#### 及时释放资源
及时释放不再使用的资源是优化的重点。在不需要使用此provider时，调用`model.dispose()`及时释放。

#### 尽可能拆分布局
减少布局重建刷新布局只刷新指定widget
