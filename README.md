## Flutter Tutorial
#### 优缺点
#### 优点

- 跨平台：Android、iOS、Fuchsia、Web、MacOS（dev分支）、Windows（快了？）
- Github活跃度非常高，更新频率高（star：91K，issue：35K）
- 2019年10月 iOS13正式版  Flutter11月推出正式版适配iOS13
- 实时跟随Android版本特性
- 渲染能力：60hz、120hz，无限接近原生
- ......

#### 缺点

- 插件市场还需完善
- 安装包大小（原生与Flutter对比）: iOS - 相差10+M ，Android - 相差6+M
- 懂一些iOS，懂一些Android，懂一些...(需要开发插件的话，就需要多懂点)
- ......

#### [原理分析](https://github.com/LiangLuDev/flutter_tutorial/blob/master/Flutter%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90.md)

#### 开发环境
[Flutter开发环境配置](https://flutter.cn/docs/get-started/install/macos)

#### [项目结构](https://github.com/LiangLuDev/flutter_tutorial/blob/master/Flutter%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84.md)

#### Android -> Flutter
- Widget - [Flutter Widget](https://github.com/LiangLuDev/flutter_tutorial/blob/master/Flutter%20Widget.md)
- 异步 - [Flutter异步](https://github.com/LiangLuDev/flutter_tutorial/blob/master/Flutter%20%E5%BC%82%E6%AD%A5.md)
- 路由 - [Flutter路由](https://github.com/LiangLuDev/flutter_tutorial/blob/master/Flutter%E8%B7%AF%E7%94%B1.md)
- 网络请求 - [Dio网络请求](https://github.com/flutterchina/dio)
- 状态管理 - [Flutter Provider](https://github.com/LiangLuDev/flutter_tutorial/blob/master/Flutter%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86-Provider.md)
#### [插件开发](https://github.com/LiangLuDev/flutter_tutorial/blob/master/Flutter%E6%8F%92%E4%BB%B6%E5%BC%80%E5%8F%91.md)
#### 打包部署
##### 参考文章
[Android打包](https://flutter.cn/docs/deployment/android)

[iOS打包](https://flutter.cn/docs/deployment/ios)

项目根目录下，执行命令
```
// iOS打包
➜  ~ flutter clean
➜  ~ flutter build ios

到xcode打包发布

// Android打包
➜  ~ flutter build apk
```
