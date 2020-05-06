
```
┬
  ├ android      - Android部分的工程文件
  ├ build        - 项目的构建输出目录
  ├ ios          - iOS部分的工程文件
  ├ images       - 存放图片资源（手动添加，名称自定）
    ┬
    └ 2.0x       - 2.0x图片资源
    └ 3.0x       - 3.0x图片资源
  ├ lib          - 项目中的Dart源文件
    ┬
    └ xxxx.dart  - 包含其他源文件
    └ main.dart  - 自动生成的项目入口文件
  ├ test         - 测试相关文件
  └ pubspec.yaml - 项目依赖配置文件
```

### 版本号设置
在`pubspec.yaml`文件
```
// 1.0.0 : version name 
// 1     : version code
version: 1.0.0+1
```

### 图片资源
在`pubspec.yaml`文件
```
flutter:
    assets:
        - images/flutter.png
```
Flutter中使用资源文件
```
//图片资源
Image.asset('images/flutter.png')
```

### 插件依赖
在`pubspec.yaml`文件中配置项目，
1. 已发布到`pub.dartlang.org`插件
```
dependencies:
    //插件名称：版本号
    cupertino_icons: ^0.1.2 
```
2. Git仓库
```
dependencies:
    cupertino_icons: 
        git：
            url：git@xxxx.com
            ref: master //分支名称
```
3. 本地
```
dependencies:
    cupertino_icons: 
        path：../cupertino_icons //本地路径
```
